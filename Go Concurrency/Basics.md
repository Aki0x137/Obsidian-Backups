# Concurrency v/s Parallelism
*Concurrency is a property of the code; parallelism is a property of the running program.*

Imagine a program with two independent tasks, like logging and core business logic, or a producer-consumer setup. These tasks could potentially run at the same time. However, if the machine only has a single CPU core, true parallel execution isn't possible. Instead, the CPU can rapidly switch between the tasks, a process known as **context switching**. For instance, if one task is waiting for an I/O operation, the CPU can work on the other task in the meantime(**interleaving** of different execution paths.).

Concurrency is about structuring code so that different parts can be executed out of order or independently. Parallelism, on the other hand, is about actually running those different parts simultaneously, which requires hardware support like multiple CPU cores or threads. Parallelism is a property of the _runtime_ of our program.

It's possible to have a distinction between concurrency and parallelism, thanks to the layers of abstraction that lie beneath our program’s model: the concurrency primitives, the program’s runtime, the operating system, the platform the operating system runs on(in the case of hypervisors, containers, and virtual machines), and ultimately the CPUs.

As we begin moving down the stack of abstraction, the problem of modeling things concurrently is becomes both more difficult to reason about, and more important.

> Go introduces goroutines and channels as primary concurrency primitives along with traditional locks. goroutines and channels are not another layer of abstraction on top of OS threads, but they **supplant** them.

Goroutines free us from having to think about our problem space in terms of parallelism and instead allow us to model problems closer to their natural level of concurrency.