# Why do we need a separate Atomic types?

- There are only certain "ways"(locks and other sync mechanisms) in which multiple threads interacting with primitive values are safe. If the primitive types like uint8, bool, etc are used and shared among thread boundaries, and we don't follow these "ways", we'll have data races and undefined behaviour. 
- One could argue to make primitives themselves thread safe, but it makes more sense to keep them separate as the underlying CPU instructions which we're issuing while working with atomics is different from how primitive types are handled.
- Atomic types puts limitations and rules on the use of the underlying value when they are exposed.