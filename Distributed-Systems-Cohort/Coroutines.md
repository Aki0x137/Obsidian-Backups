A **coroutine** is essentially a function that can pause its execution and resume later from where it left off. This is different from regular functions, which run to completion in one go.

### Key Characteristics:
- **Suspending and Resuming:** Coroutines can voluntarily yield control back to the caller and later resume execution at the same point.
- **Lightweight:** Compared to threads, coroutines are generally more lightweight and efficient.
- **Cooperative Multitasking:** Coroutines rely on each other to yield control, unlike threads which are managed by the operating system.