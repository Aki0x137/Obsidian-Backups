A **coroutine** is essentially a function that can pause its execution and resume later from where it left off. This is different from regular functions, which run to completion in one go.

### Key Characteristics:
- **Suspending and Resuming:** Coroutines can voluntarily yield control back to the caller and later resume execution at the same point.
- **Lightweight:** Compared to threads, coroutines are generally more lightweight and efficient.
- **Cooperative Multitasking:** Coroutines rely on each other to yield control, unlike threads which are managed by the operating system.

Coroutines are particularly useful for:
- **Asynchronous programming:** Handling tasks that might take a long time without blocking the main thread.
- **Iterators:** Creating custom iterators with complex logic.
- **Cooperative multitasking:** Managing multiple tasks concurrently without the overhead of threads.