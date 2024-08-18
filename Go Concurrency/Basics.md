# Concurrency v/s Parallelism
*Concurrency is a property of the code; parallelism is a property of the running program.*

Imagine a program with two independent tasks, like logging and core business logic, or a producer-consumer setup. These tasks could potentially run at the same time. However, if the machine only has a single CPU core, true parallel execution isn't possible. Instead, the CPU can rapidly switch between the tasks, a process known as **context switching**. For instance, if one task is waiting for an I/O operation, the CPU can work on the other task in the meantime(**interleaving** of different execution paths.).

Concurrency is about structuring code so that different parts can be executed out of order or independently. Parallelism, on the other hand, is about actually running those different parts simultaneously, which requires hardware support like multiple CPU cores or threads. Parallelism is a property of the _runtime_ of our program.