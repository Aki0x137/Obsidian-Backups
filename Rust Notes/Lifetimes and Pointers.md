Lifetimes are regions of code, where certain references must be valid for.

**Liveness**: A variable is live if its current value may be used later in the program.

when a reference is valid and what they point to a particular time is what fully defines lifetime of a reference.

Reference is just a special type of pointer. Special because it has the `'a` thing, that the compiler uses to validate that the reference never points to invalid memory. Usually we think of `'a` as a reference's lifetime, as in which regions of code the reference needs to be valid in.
We need to think of both: when and where the reference is valid, and what it points to, as in which region of memory.

The lifetime annotations `'a` basically tells compiler, that this function should be passed parameters(which are references) which satisfy the relationship, specified using the annotations. If the references passed do not satisfy the relationship, throw error at compile time.