# Concurrency v/s Parallelism
*Concurrency is a property of the code; parallelism is a property of the running program.*

Imagine a program with two independent tasks, like logging and core business logic, or a producer-consumer setup. These tasks could potentially run at the same time. However, if the machine only has a single CPU core, true parallel execution isn't possible. Instead, the CPU can rapidly switch between the tasks, a process known as **context switching**. For instance, if one task is waiting for an I/O operation, the CPU can work on the other task in the meantime(**interleaving** of different execution paths.).

Concurrency is about structuring code so that different parts can be executed out of order or independently. Parallelism, on the other hand, is about actually running those different parts simultaneously, which requires hardware support like multiple CPU cores or threads. Parallelism is a property of the _runtime_ of our program.

It's possible to have a distinction between concurrency and parallelism, thanks to the layers of abstraction that lie beneath our program’s model: the concurrency primitives, the program’s runtime, the operating system, the platform the operating system runs on(in the case of hypervisors, containers, and virtual machines), and ultimately the CPUs.

As we begin moving down the stack of abstraction, the problem of modeling things concurrently is becomes both more difficult to reason about, and more important.

> Go introduces goroutines and channels as primary concurrency primitives along with traditional locks. goroutines and channels are not another layer of abstraction on top of OS threads, but they **supplant** them.

- Goroutines free us from having to think about our problem space in terms of parallelism and instead allow us to model problems closer to their natural level of concurrency. 
- Channels, for instance, are inherently *composable* with other channels. This makes writing large systems simpler because you can coordinate the input from multiple subsystems by easily composing the output together. Coordinating mutexes is a much more difficult proposition.
- The `select` statement is the complement to Go’s channels and is what enables all the difficult bits of composing channels. `select` statements allow you to efficiently wait for events, select a message from competing channels in a uniform random way, continue on if there are no messages waiting, and more.

*Do not communicate by sharing memory. Instead, share memory by communicating*.

![[Pasted image 20240818200222.png]]

> If you find yourself exposing locks beyond a type, this should raise a red flag. Try to keep the locks constrained to a small lexical scope.

> If you find yourself struggling to understand how your concurrent code works, why a deadlock or race is occurring, and you’re using primitives, this is probably a good indicator that you should switch to channels.

> Remember that channels are inherently more composable than memory access synchronization primitives. Having locks scattered throughout your object-graph sounds like a nightmare, but having channels everywhere is expected and encouraged

Go’s philosophy on concurrency can be summed up like this: aim for simplicity, use channels when possible, and treat goroutines like a free resource.