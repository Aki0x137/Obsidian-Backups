In concurrent programming, work scheduling is crucial for efficiently managing tasks running simultaneously on multiple processors.

A **work scheduler** acts like a traffic controller, distributing **concurrent tasks** among available processors. Its goal is to:
- **Maximize Throughput:** Get as much work done as possible in the least time.
- **Balance Load:** Ensure processors are evenly utilized, avoiding idle time on some while others are overloaded.
- **Minimize Overhead:** Scheduling itself should be efficient, not creating a bottleneck.
#### Common Strategies:
Different scheduling algorithms exist, each with its strengths and weaknesses. They broadly fall in below categories:
- **Static Scheduling:** Assigns tasks to processors beforehand, good for predictable workloads.
- **Dynamic Scheduling:** Assigns tasks on-the-fly, better for workloads with unknown execution times.
- **Priority Scheduling:** Prioritizes certain tasks for faster execution.
For example, TensorFlow used Static Scheduling, where users/programmers had to create DAG. Whereas, PyTorch used Dynamic Scheduling, where the DAG was created at runtime.
## Why do we need to understand this?
- We can still write parallel code without work scheduler's knowledge.
- However, knowing the basics of work stealing will make you a better programmer.
- We want to know, if we can divide a task in small chunk, what strategies can we use so that our multi core CPU can be used as efficiently as possible?
## Parallel Scheduling
We want to build a parallel scheduler, where we can:
- efficiently allocate cores to tasks in a computation(Task can be different things, like threads, coroutines, function, etc).
- keep context-switches minimum.
- Prevent loss of data locality.
- Basically, keep the CPU cores completely busy, executing parallel tasks in any order.

> [!INFO] Context Switching & Scheduling
> - Context switching involves saving the state of the current task and loading the state of the next one.
> - Unlike concurrency, parallelism doesn't benefit from fair scheduling. On the contrary, fair scheduling tends to increase context switching.
> - In **parallel systems**, frequent context switching can become a bottleneck.
> - It adds overhead because the processor spends time managing tasks instead of actually executing them in parallel.
> 
> - However, in **concurrent systems**, fair scheduling is often desirable. Fair scheduling ensures that all tasks get a fair share of the CPU time, preventing any single task from starving. This is crucial in systems where tasks might have different priorities or where user-facing responsiveness is critical.
> -  Fair scheduling involves frequent context switching, where the CPU switches between tasks to give each one a turn. This approach increases overhead due to the cost of saving and restoring task states.

- Below is a diagram, explaining typical, work scheduling flow for parallel computation. It is essentially a DAG:

![[Excalidraw/Parallel Computation DAG|{width=100%}]]

### Types of Schedulers
1. **Greedy Schedulers:** 
	- **Goal:** Make the "best" choice at each step to optimize a specific objective, typically minimizing completion time or maximizing resource utilization. With a greedy scheduler, no processor is idle if there is more work it can do.
	-  **Focus:** Short-term gain. They prioritize the most beneficial option at each step without considering the long-term consequences.
	- **Example:** Shortest Job First (SJF) scheduling aims to pick the job with the shortest execution time next, potentially leading to starvation for longer jobs if a stream of short jobs arrives continuously.
1. **Non-Greedy Schedulers:**
	 - **Goal:** Don't necessarily make the "best" choice at every step, but focus on finding an acceptable solution that avoids major drawbacks.
	 - **Focus:** Balance between short-term and long-term goals. They may sacrifice some immediate optimization for overall fairness or stability.
	 - **Example:** Round Robin scheduling gives each job a fixed time slice to run, ensuring all jobs eventually get a chance to progress, even if it doesn't minimize the overall completion time for all jobs.
	
> [!Note] Time Complexity
> Computing the optimum schedule is NP-Complete, even if the complexity of each task is known in advance.
### Design Constraints on Work Scheduler
1. The DAG should be generated dynamically:
   - Based on inputs and programs control flow
   - The graph is not known ahead of time
2. The amount of work done by a task is dynamic
3. The number of processors can change at runtime
4.  Should be greedy in nature:
	- No processor should be idle when tasks are pending
5. Limited communication between processors
	-  due to the overhead that comes with it.
6. If possible, should schedule related tasks in the same processor
	- Access same cache lines
- Point 4 & 5 is important for us.
#### Approach 1: Manager/Centralized Scheduler
1. We can have a manager that serves as a centralized scheduler running on one processor.
2. The manager distributes task to other processors.
3. Simple to implement, provides centralized control.
**Problems**:
1. **Synchronization bottleneck**: Frequent communication with the manager can create a bottleneck, slowing down overall performance.
2. **Single point of failure:** If the manager fails, the entire system stalls.
#### Approach 2: Shared Work Queue
1. All processors share a common work queue
2. Every processor dequeue a task from work queue
3. On task completion, enqueue the generated task to the work queue
**Problems**:
1. **Synchronization overhead:** Accessing and modifying the shared queue requires synchronization, impacting performance.
2. **Cache locality issues**: asks may not be located in the cache of the processor fetching them, leading to more main memory access and slower execution.
3. **Unbounded queue size**: The queue can potentially grow infinitely, leading to memory limitations.
#### Approach 3: Work sharing (Push-Based)
1. Every processor pushes and pops tasks into a local work queue
2. When the work queue gets large, send tasks to other processors
**Problems**:
1. **Busy communication**: If all processors are busy, each will spend time trying to offload extra tasks.
	1. This means they will try to perform communication when busy, which will violate constraint 5
2. **Load imbalance**: It's difficult to know the load on processors.
	- A processor with two large tasks might take longer than a processor with five small tasks.
	- Each processor only has information about its own workload and the size (not necessarily the execution time) of tasks in its queue. It doesn't have a complete picture of the workload on other processors.
3. **Task Bouncing**: A task might get shared multiple times by multiple bust processors before being executed, until an available processor is found.
4. **Non-greedy**: The Approach is *non greedy*, some processors can be idle for some time, waiting for a processor to share some work.

The next section will cover the desired approach, which is work stealing.

### Work Stealing
Work stealing is a popular approach for parallel work scheduling that addresses the limitations of previous approaches discussed.
#### Key aspects:
1. **Only the idle workers steal**:
	- In a work-stealing scheduler, workers (or processors) that run out of work become idle. Instead of waiting, these idle workers actively seek tasks from other busy workers to keep themselves productive.
2. **Each processor maintains a local work queue**: 
	- Each processor has its own local queue to store tasks it needs to execute. This helps reduce contention since most of the time, processors operate independently on their local queues.
3. **Pushes generated tasks into the local queue**:
	- When a processor creates new tasks, it pushes them onto its local queue. This ensures that the processor can continue working on related tasks, leveraging data locality.
4. **When local queue is empty, steal a task from another processor**:
	- If a processor’s local queue is empty, it attempts to steal tasks from the queues of other processors. This ensures that no processor remains idle if there are still tasks left to execute.

**Pros**:
- **Communication is only done when idle**: Communication between processors happens only when a processor is idle and needs to steal tasks. This minimizes unnecessary communication overhead when all processors are busy.
	- There is little to no communication when all processors are in full throttle.
- **Each task is stolen at most once**: Once a task is stolen by a processor, it’s removed from the original queue, ensuring that no other processor steals it again, thus avoiding duplicate work.
- This scheduler is greedy, assuming stealing succeeds always
- **Tasks are identified dynamically, as execution proceeds**: Tasks are not pre-allocated statically. Instead, they are dynamically identified and assigned as the program runs, allowing for flexibility in task distribution.
- **Tasks are performed by a (usually fixed-size) pool of workers**: A predefined number of workers execute the tasks. This fixed pool simplifies the management of resources.
	- At the hardware level, a worker corresponds to a CPU core.
	- At the software level, workers are usually represented by threads, one per core.
- Any worker can 'steal' a task that was created by another worker
- **Dynamic load balancing (near greedy)**: Work distributed automatically from workers that have unstarted tasks to workers that have no tasks.
	- Work stealing (and parallelism in general) require that work be divided into independent tasks.
	- Once a worker begins a task, it cannot be shared with other workers unless it is subdivided into smaller tasks.
	
**Queue Data Structure**: A special double ended queue is used to maintain the local task queue for each worker. It has the following operations:
1. Push: Local processor adds newly created tasks
2. Pop: Local processor removes tasks to execute
3. Steal: Remote process removes task
Advantages of using queue:
- Stealers don't interact with local processor when queue has more than one tasks
	- Minimizing interaction reduces contention and overhead, improving efficiency.
- Popping recently pushed tasks improves locality:
	- Accessing recently added tasks can benefit from cache locality, speeding up execution.
- Stealers take the oldest tasks, which are likely to be the largest:
	- Larger tasks are more worthwhile for stealing because they provide more work, reducing the frequency of stealing operations.



