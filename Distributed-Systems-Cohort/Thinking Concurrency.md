This note tries to create mental model for concurrency and parallelism. These mental models extrapolate(have a 1:1 mapping) well, when we talk abut distributed system.
Concepts like message passing, consistency, consensus are common in both concurrent programs and distributed systems.

Few things before we move ahead:
- Anything discussed with respect to distributed systems will be around, **networked multi-core servers**.
- We have cluster/racks of these kind of servers, and each node in this system is connected(networked).
# Why are we discussing parallel and high performance software?

## Moore's Law:
Moore's Law stated that the number of transistors on a microchip would double roughly every two years, leading to exponential growth in processing power.

So, the same software written in let's say in year 1999, would perform better, when it runs on a better processor, created in 2010. Improvement in single thread performance(hardware improvements), led to automatic improvement in performance of software without any change. This phenomenon of system's ability to handle growing demands by adding resources or improvement is underlying hardware it runs on is called Architectural scalability.

However, Moore's Law held true till last decade(early 2020s) but is now nearing its physical limits. Transistors are approaching atomic size, making further miniaturization difficult. With slower growth in core speed, developers can no longer solely rely on Moore's Law for performance gains.

![[Screenshot from 2024-07-14 13-15-29.png]]

So, software engineers should now focus on techniques like code optimization, parallel programming, and utilizing multiple cores effectively. By squeezing the most out of current hardware, they can achieve performance improvements that were previously attained through Moore's Law alone.

We have to change the way we think about programs, and ask what are the primitives and software constructs, which will change, when I want my program to make use of multi core in the chip available to me.
## Why is Concurrency hard?
- Multiprocessor programming is complex due to the inherent asynchrony of modern computer systems.
- Async events, like *interrupts, preemption, cache misses and failures* can halt or delay activities without warning.
- These delays are unpredictable and can very widely:
	- A cache miss might delay a processor for few instructions.
	- A page fault can delay it got millions of instructions.
	- OS preemption can result in delays spanning hundreds of millions of instructions.
Example: https://vaibhaw-vipul.medium.com/matrix-multiplication-optimizing-the-code-from-6-hours-to-1-sec-70889d33dcfa

## Concurrency v/s Parallelism
**Concurrency** focuses on managing multiple tasks that can potentially run at the same time, even if they share processing power (like multitasking on a single CPU). 
**Parallelism** explicitly utilizes multiple processors or cores to execute tasks truly simultaneously.
TODO: Watch Rob Pike's video on same and write summarisation here

# Mutual Exclusion
In concurrent programming, mutual exclusion is a fundamental concept for ensuring data integrity. It refers to the idea that only one thread of execution can access a shared resource(like a piece of memory, critical section in code or a file) at a time.
This prevents race conditions, where multiple threads try to modify the same data simultaneously, potentially leading to unexpected or incorrect results.
### How do we make sure our protocol or algorithm is correct?
#### 1. Mutual Exclusion:
- Threads are prevented from accessing the same resource(memory address or critical region in code) simultaneously.
#### 2. Deadlock-Freedom:
- If a thread want to access a resource, it eventually succeeds.
- If two or more threads want to access a resource, at least one succeeds.
#### 3. Starvation Freedom:
- Whether a thread can eventually access the shared resource.
#### 4. Waiting:
- Threads make progress within a bounded number of steps, irrespective of the activity from other threads.

## Harsh Realities of Parallel Programming

### Amdahl's Law:
- A principle that states that the maximum potential improvement to the the performance of a system is limited by the portion of the system/code that cannot be parallelised or improved.
- Imagine a task with sequential (non-parallelizable) parts(around 40%) and parallelizable parts(rest 60%). Amdahl's Law lets you calculate the maximum speedup achievable by adding more processors, considering the percentage of time spent on the sequential portion. In this case 60% of code will never improve by adding more processors, and only the 40% code can be parallelised and optimised.
- In simpler terms, the more a task relies on unparallelizable steps, the less benefit you get from adding more processing power.
- The speedup factor refers to how much faster a program runs using multiple processors compared to running it on a single processor.
```
  Speedup Factor =  s(p) = t(s) / t(p)
``` 
where: 
- p = Number of processors
- T(s) = execution time on one processor 
- T(p) = execution time using p processors.

- There are several factors that affect the speedup:
	1. periods of busy v/s the idle times in processor
	2. some extra constants that need to be calculated again and again
	3. communication time
- Lets assume `f` the fraction of computation which can't be divided into concurrent task, an no overhead is incurred when tasks are divided.
- The time to perform computation with `p` processors is given by: 
  `f*t(s) + {[(1 - f)*t(s)] / p} ` 
- The speedup factor becomes:
```
Speedup Factor =  s(p) = t(s) / { f*t(s) + {[(1 - f)*t(s)] / p} }

= p / [1 - (p - 1)*f]
```
The above equation is mathematical representation of Amdahl's law.

![[Pasted image 20240714153535.png]]
### Gustafson's Law:
- Amdahl's law doesn't take into account **Algorithmic Scalability** though.
- When working in a multiprocessing world, we also have to think from algorithm side. Algorithm also need to incorporate 'n' processors.
- Combined Architectural Scalability and Algorithmic scalability suggests increased problem size(larger inputs in problem statement) can be accommodated with increase in system size for particular architecture and algorithm.
- For example, for a particular algorithm, let's say matrix multiplication, we have:
	- n = input size
	- p = number of processors
- Both n and p matter. Altering p alters size of computer, and altering n alters size of problem.
- Usually increasing problem size, with a fixed number of processors, improves relative performance(until a certain threshold), because more parallelism is achieved.
- Gustafson argued, usually `f`, i.e. the serial section of the algorithm/code is normally fixed, and doesn't increase with problem size `n`.

- Gustafson's Law acknowledges that with more processors, we can tackle larger problems. This allows the parallelizable portion of the workload to scale, potentially leading to greater speedup than Amdahl's Law might predict for a fixed-size problem.
- Gustafson's Law suggests that by throwing more processing power at problems that can be effectively parallelised and also scaled in size, we can achieve significant performance gains even with inherent sequential portions.
```
Speedup Factor =  s(p) = p - f(p - 1)
```
- p = Number of processors
- f = The fraction of the workload that is inherently sequential (unchangeable)