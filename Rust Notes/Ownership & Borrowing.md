# Ownership

 - Deals with how a memory reference is managed in a program.
 - Any program when executing, stores data in two different parts of computer memory with different guarantees:
	 - Stack: Stores local variables, that have scope limited to a function execution or block, and whose size is known during compile time.
	 - Heap: Stores global variables or values that need to be in existence till the end of program, or whose size is dynamic.
 - In any program, written in Rust or any other language, a variable that stores data with a **known** and **fixed** size is stored in stack. For example `let x: i32 = 5;` will take 32 bits of memory in our program. Here, both the variable and the value, will be stored in stack.

> [!NOTE] Stack vs Heap
> - Stack supports fast memory creation and retrieval.
> - Memory occupied by unused code is automatically recaptured by the program after it goes out of the scope.
> - Heaps on the other hand, provide flexibility in terms of memory allocations. Values that can grow in size, like vectors, hash maps, strings, etc, live on heap.
> - heaps also allow for memory(and thus values) to live beyond the scope that created it.
> - Memory used by values in heap are recaptured when the last owner goes out of scope. 


 - Lets consider below code:
```rust
let mut vec1 = vec![1, 2, 3];
vec1.push(4);
println!("{:?}", vec1);
drop(vec1);
```
- Vector is declared a *mutable*. So it can change or its size can change when the program is running. So, the vector object is stored in stack and the actual content(value) of the vector is stored in heap.
- When we drop the variable,  both *the* data on stack and heap(the value and the object/var) will not be accessible.
 - For programs that use pointers, this could be problematic since, two or more variables can "point" to same memory location.
 - Lets say in a C/C++ program, a memory location is referenced by 2 variables. Now accidentally, you delete/free one of the variables which also frees the memory location.
   The other pointer will become a "dandling pointer", which can cause program to crash if trying to de-reference the pointer.
- Rust solves this problem by ownership rules. These set of rules are verified by compiler at compile time. 

- Lets consider another example:
```rust
let mut say = String::from("Indi");
say.push_str("a");

say_what = say;

println!(say);
```

> [!info] Rules of Ownership:
> 1. Each value in Rust is associated with / assigned to a variable. This variable is called the **owner** of the value. In above example code, `say` is the owner of the string.
> 2. There can be **only one owner** of a value at *heap*, at a time. In above example, when we assign *value* of `say` to `say_what` in line 3, the ownership of string *moves* to `say_what`, and invalidates the ownership of `say`.  Line 4 therefore, result in error, since variable `say` doesn't has ownership of the value.
> 3. When the owner goes **out of scope**, the value will be **dropped**. 

- The ownership prevents below safety issues:
	1. Dangling pointers
	2. Double-free: Trying to free memory that has already been freed
	3. Memory leaks: Not freeing memory that should have been freed

# Borrowing

## Scopes
- Scope is the **range** within a program, for which that **variable** and the **value** are **valid/accessible**.
- scopes define the region of code within which variables, types, functions, and other constructs are valid.
- Lets consider below example:

```rust
fn print_out(to_print: String) {
	println!("{}", to_print);
}

fn main() {
	let say = String::from("Cat");
	print_out(say);

	println!("Again {}", say);
}
```

- When we pass, a value to function(in the above code `say` to `print_out`), we transfer ownership to that function.
- Once we transfer ownership, the value is out of scope, in the callee.
- If we try to access the value of the transferred/moved, we get error, since the value is no longer **valid**.

- But we do need to pass values around functions. To do that, we need to tell the compiler, how exactly the value will be used.
- In other languages, we can pass data around functions using 2 methods: pass by value and pass by reference.
- In Rust, if just need to send a copy, we explicitly clone the value and send it. This will create a copy of the value in stack or heap(depending on mutability). Example:
```rust
fn print_out(to_print: String) {
	println!("{}", to_print);
}

fn main() {
	let say = String::from("Cat");
	print_out(say.clone());

	println!("Again {}", say);
}
```
- Cloning is expensive though, since you create new copy every time.

- If we want to send the same underlying value, we pass a reference of the value. Example:
```rust
fn print_out(to_print: &String) {
	println!("{}", to_print);
}

fn main() {
	let say = String::from("Cat");
	print_out(&say);

	println!("Again {}", say);
}
```

- When we pass by reference, we say the called function is **borrowing** the value.

> [!info]  Rules of Borrowing:
> 1. At any given time, You can have either:
> 	- one mutable reference, or
> 	- any number of immutable references.
> 2. References must always be valid.

- References by default are immutable, but can be made mutable using `mut` keyword.
- Example:
```rust
fn add_to_vec(a_vec: &mut Vec<i32>) {
	a_vec.push(4);
}

fn main() {
	let mut my_vec = vec![1, 2, 3];
	println!("{:?}", my_vec);

	add_to_vec(&mut my_vec);
	println!("{:?}", my_vec);
}

// Output:
// [ 1, 2, 3 ]
// [ 1, 2, 3, 4 ]
```

// More notes from Doug Milford video