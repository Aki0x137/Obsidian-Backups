## Why do we need to understand this?
- We can still write parallel code without work scheduler's knowledge.
- However, knowing the basics of work stealing will make you a better programmer.
- We want to know, if we can divide a task in small chunk, what strategies can we use so that our multi core CPU can be used as efficiently as possible?
## Parallel Scheduling
We want to build a parallel scheduler, where we can:
- efficiently allocate cores to tasks in a computation.
- keep context-switches minimum.
- Prevent loss of data locality.
- Basically, keep the CPU cores completely busy, executing parallel tasks in any order.
- Unlike concurrency, parallelism doesn't benefit from fair scheduling. On the contrary, fair scheduling tends to increase context switching.
- Below is a diagram, explaining typical, work scheduling flow for parallel computation. It is essentially a DAG:
![[Excalidraw/Parallel Computation DAG]]
### Design Constraints on Work Scheduler
1. The DAG should be generated dynamically:
	-  Based on inputs and programs control flow
	- The graph is not known ahead of time
1. The amount of work done by a task is dynamic
2. The number of processors can change at runtime