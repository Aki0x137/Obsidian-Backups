This note tries to create mental model for concurrency and parallelism. These mental models extrapolate(have a 1:1 mapping) well, when we talk abut distributed system.
Concepts like message passing, consistency, consensus are common in both concurrent programs and distributed systems.

Few things before we move ahead:
- Anything discussed with respect to distributed systems will be around, **networked multi-core servers**.
- We have cluster/racks of these kind of servers, and each node in this system is connected(networked).
# Why are we discussing parallel and high performance software?

## Moore's Law:
Moore's Law stated that the number of transistors on a microchip would double roughly every two years, leading to exponential growth in processing power.

So, the same software written in let's say in year 1999, would perform better, when it runs on a better processor, created in 2010. Improvement in single thread performance(hardware improvements), led to automatic improvement in performance of software without any change.

However, Moore's Law held true till last decade(early 2020s) but is now nearing its physical limits. Transistors are approaching atomic size, making further miniaturisation difficult. With slower growth in core speed, developers can no longer solely rely on Moore's Law for performance gains.

![[Screenshot from 2024-07-14 13-15-29.png]]

So, software engineers should now focus on techniques like code optimisation, parallel programming, and utilising multiple cores effectively. By squeezing the most out of current hardware, they can achieve performance improvements that were previously attained through Moore's Law alone.

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


# Mutual Exclusion





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

