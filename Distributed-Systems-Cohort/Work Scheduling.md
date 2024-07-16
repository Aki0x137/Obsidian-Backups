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
### Design Constraints on Work Scheduler
1. The DAG should be generated dynamically:
   - Based on inputs and programs control flow
   - The graph is not known ahead of time
2. The amount of work done by a task is dynamic
3. The number of processors can change at runtime
4. Should be greedy in nature:
	- No processor should be idle when tasks are pending
5. Limited communication between processors
	-  due to the overhead that comes with it.
6. If possible, should schedule related tasks in the same processor
	- Access same cache lines

