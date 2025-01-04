# Ownership
 - Deals with how a memory reference is managed in a program.
 - Any program when executing, stores data in two different regions of computer volatile memory with different guarantees:
	 - **Stack**: Stores local variables, that have scope limited to a function execution or block, and whose size is known during compile time.
	 - **Heap**: Stores global variables or values that need to be in existence till the end of program, or whose size is dynamic.
	 - **Static**: Stores binary instructions of program and static variables.
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
- When we drop the variable, both *the* data on stack and heap(the value and the object/var) will not be accessible.
 - For programs that use pointers, this could be problematic since, two or more variables can "point" to same memory location.
 - Lets say in a C/C++ program, a memory location is referenced by 2 variables. Now accidentally, you delete/free one of the variables which also frees the memory location.
   The other pointer will become a "dandling pointer", which can cause program to crash if trying to de-reference the pointer.
- Rust solves this problem by ownership rules. These set of rules are verified by compiler at compile time. 

- Lets consider another example:
```rust
let mut say = String::from("Indi");
say.push_str("a");

say_what = say;

println!(say); // will cause compiler error
```

> [!info] Rules of Ownership:
> 1. Each value in Rust is associated with / assigned to a variable. This variable is called the **owner** of the value. In above example code, `say` is the owner of the string.
> 2. There can be **only one owner** of a value at *heap*, at a time. In above example, when we assign *value* of `say` to `say_what` in line 3, the ownership of string *moves* to `say_what`, and **invalidates** the ownership of `say`.  Line 4 therefore, result in error, since variable `say` doesn't has ownership of the value.
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
- Once we transfer ownership, the value is out of scope, in the caller.
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
**Borrowing** is:
- Establishing a reference to some data in memory
	- it is like pointers with added rules
	- it does **not** take ownership
> [!info]  Rules of Borrowing:
> 1. At any given time, You can have either:
> 	- one mutable reference, or
> 	- any number of immutable references.
> 2. References must always be valid.

These rules solve the problem of **data races** and **dandling references**.

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
#### Dangling References (Prevented by Rust)
Rust ensures that references are never dangling.
```rust
fn main() {
    let r;
    {
        let x = 5;
        r = &x; // Error: x does not live long enough
    }
    println!("{}", r); // !!! r would point to invalid memory
}
```

The below code will not compile because, `some_vec` 's `&mut self` reference is passed in `push` while an immutable reference is still in scope and passed to `println` and `get_first_element`.
```rust
fn main() {
	let mut some_vec = vec![1, 2, 3];
	let first = get_first_element(&some_vec);
	some_vec.push(4);
	println!("The first number is: {}", first);
}

fn get_first_element(num_vec: &Vec<i32>) -> &i32 {
	&num_vec[0]
}
```
## Special Cases
### Borrowing and Mutability
Even if a value is mutable, an immutable reference can still be created as long as there are no active mutable references.
```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // Immutable reference
    println!("{}", r1); // r1 used here
    let r2 = &mut s; // Mutable reference allowed after r1 is done
    r2.push_str(", world");
    println!("{}", r2);
}
```
### Reborrowing
Rust allows **reborrowing** of references.
```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &*r1; // Reborrowing as immutable reference
    println!("{}", r2);
}
```
Mutable reference(`& mut`) cannot be copied, but can only be moved.
- References reside in stack.
- Stack allocated data, when assigned to a new variable, are copied and assigned to new variable. The ownership is not moved.
- However, mutable references, which themselves are stored on stack, is an exception.
- This is done to ensure conformity with Rust borrowing rule.
	- According to the rules of borrowing discussed above, we cannot have multiple mutable references.
example:
```rust
let mut heap_data = vec![5, 6, 7];
let ref_1 = &mut heap_data;

let move_ref_out = ref1; // mutable reference `move`d to move_ref_out. ref_1 will no longer be valid
let move_ref_again = ref_1; // error out, trying to create another mutable ref from ref_1 when it longer is valid.
```
> The above code would have worked if all of them were immutable references. Also, immutable references are copied and not moved, so ref_1 would still hold a reference.
# `&mut self` and `&self` 
In Rust, the usage of `&self`, `&mut self`, and their relationship with references revolves around **ownership**, **borrowing**, and **mutability** when implementing methods for structs or enums. Let’s delve into their meanings and nuances.
## `&self` and `&mut self` in Method Definitions
### 1. `&self`: Immutable Borrow
- When a method takes `&self`, it borrows the instance of the type immutably.
- This means the method can **read** but not **modify** the instance.
```rust
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    // Method with an immutable borrow
    fn get_coordinates(&self) -> (i32, i32) {
        (self.x, self.y) // Can access fields but cannot modify them
    }
}

fn main() {
    let p = Point { x: 3, y: 5 };
    let coords = p.get_coordinates(); // p is immutably borrowed
    println!("Coordinates: {:?}", coords); // p is still valid here
}
```
### 2. `&mut self`: Mutable Borrow
- When a method takes `&mut self`, it borrows the instance **mutably**.
- This allows the method to both **read** and **modify** the instance.
```rust
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    // Method with a mutable borrow
    fn move_by(&mut self, dx: i32, dy: i32) {
        self.x += dx;
        self.y += dy; // Modifying the instance
    }
}

fn main() {
    let mut p = Point { x: 3, y: 5 }; // p must be mutable
    p.move_by(2, 3); // p is mutably borrowed
    println!("Moved Point: ({}, {})", p.x, p.y);
}
```
### Why Distinction Matters
1. **Ownership and Borrowing Rules**:
    - A `&self` method allows multiple calls simultaneously because it doesn't modify the object.
    - A `&mut self` method requires exclusive access to the object while it is being called.
2. **Safety**:
    - Rust enforces that immutable and mutable borrows cannot coexist, preventing data races.
## Reference Behavior in `&self` and `&mut self`
### 1. Passing `self` by Reference
Both `&self` and `&mut self` are syntactic sugar for explicit references to `self`:
- `fn method(&self)` is shorthand for `fn method(self: &Self)`.
- `fn method(&mut self)` is shorthand for `fn method(self: &mut Self)`.
This means `self` is a **reference** to the object, not the object itself.
```rust
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    // Explicit reference
    fn get_coordinates(self: &Self) -> (i32, i32) {
        (self.x, self.y)
    }
    fn move_by(self: &mut Self, dx: i32, dy: i32) {
        self.x += dx;
        self.y += dy;
    }
}
```
### 2. Ownership of `self`
You can also define methods that take ownership of `self` instead of borrowing it. These are useful when you want the method to consume the instance.
```rust
impl Point {
    fn into_tuple(self) -> (i32, i32) {
        (self.x, self.y) // Takes ownership of self
    }
}

fn main() {
    let p = Point { x: 3, y: 5 };
    let tuple = p.into_tuple(); // p is consumed and no longer valid
    println!("Point as tuple: {:?}", tuple);
}
```
## Reference Mutability Rules with `&self` and `&mut self`
### Immutable References (`&self`)
Multiple immutable references can coexist safely because no modifications are allowed.
```rust
fn main() {
    let p = Point { x: 3, y: 5 };

    let r1 = &p;
    let r2 = &p;

    println!("r1: ({}, {}), r2: ({}, {})", r1.x, r1.y, r2.x, r2.y);
    // Both r1 and r2 can access `p` at the same time.
}
```
### Mutable References (`&mut self`)
Only one mutable reference is allowed at a time.
```rust
fn main() {
    let mut p = Point { x: 3, y: 5 };

    let r1 = &mut p;
    // let r2 = &mut p; // Error: cannot borrow `p` as mutable more than once
    r1.x += 1;
    println!("Updated r1: ({}, {})", r1.x, r1.y);
}
```
### Immutable and Mutable References Cannot Coexist
Rust prevents simultaneous mutable and immutable references to ensure safety.
```rust
fn main() {
    let mut p = Point { x: 3, y: 5 };

    let r1 = &p;          // Immutable borrow
    // let r2 = &mut p;   // Error: cannot borrow `p` as mutable because it is already borrowed immutably
    println!("Immutable r1: ({}, {})", r1.x, r1.y);
}
```
## Use Cases for `&self`, `&mut self`, and `self`
1. **Use `&self`**:
    - When the method does not modify the instance.
    - Example: `fn len(&self) -> usize`.
2. **Use `&mut self`**:
    - When the method modifies the instance.
    - Example: `fn push(&mut self, value: T)`.
3. **Use `self`**:
    - When the method consumes the instance (e.g., transforms it or moves its contents elsewhere).
    - Example: `fn into_vec(self) -> Vec<T>`.
## Advanced Nuances
### 1. Chaining Mutable Methods
Mutable methods can be chained if they return `&mut self`:
```rust
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn move_x(&mut self, dx: i32) -> &mut Self {
        self.x += dx;
        self
    }
    fn move_y(&mut self, dy: i32) -> &mut Self {
        self.y += dy;
        self
    }
}

fn main() {
    let mut p = Point { x: 0, y: 0 };

    p.move_x(5).move_y(3); // Method chaining
    println!("Point: ({}, {})", p.x, p.y);
}
```
### 2. Using `self` with Closures
Passing `self` into closures requires understanding borrowing vs. ownership:
```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 3, y: 5 };

    let closure = move || {
        println!("Point: ({}, {})", p.x, p.y);
    }; // `p` is moved into the closure
    closure();
}
```
# Dereferencing
Dereferencing in Rust involves accessing the data that a reference or pointer points to. Rust has strict and well-defined rules for dereferencing, ensuring safety while allowing you to work with references and smart pointers.
In Rust, this is done using the **dereference operator** (`*`).
```rust
fn main() {
    let x = 5;
    let y = &x; // y is a reference to x

    println!("y: {}", *y); // Dereferencing y to access the value of x
}
// y: 5
```
## Types of References and Pointers in Rust
1. **References** (`&T` and `&mut T`): Borrowed references.
2. **Raw Pointers** (`*const T` and `*mut T`): Unsafe, low-level pointers.
3. **Smart Pointers**: Abstractions over raw pointers, like `Box<T>`, `Rc<T>`, or `Arc<T>`.
## 1. Dereferencing Safe References
### Immutable References (`&T`)
- You can dereference an immutable reference to **read** the value it points to.
- Rust guarantees that the data being referred to is valid and will not be modified during the lifetime of the reference.
```rust
fn main() {
    let x = 42;
    let ref_x = &x; // Immutable reference to x

    println!("Value through ref_x: {}", *ref_x); // Dereferencing ref_x
}
```
#### Key Rules:
- You can have **multiple immutable references** to the same value at the same time.
- You **cannot modify** the value through an immutable reference.
### Mutable References (`&mut T`)
- You can dereference a mutable reference to **read** or **write** the value it points to.
- Rust enforces that there is **only one mutable reference** to a value at a time to ensure no data races.
```rust
fn main() {
    let mut x = 42;
    let ref_x = &mut x; // Mutable reference to x

    *ref_x += 1; // Dereferencing and modifying the value
    println!("Updated value: {}", x); // Prints 43
}
```
#### Key Rules:
- You can only have **one mutable reference** to a value at a time.
- A mutable reference cannot coexist with immutable references to the same value.
### Immutable vs. Mutable Reference Rules
Rust prevents mutable and immutable references from coexisting:
```rust
fn main() {
    let mut x = 42;

    let r1 = &x; // Immutable reference
    // let r2 = &mut x; // Error: Cannot borrow as mutable while immutable references exist

    println!("Immutable reference: {}", r1);
}
```
This ensures **memory safety** and prevents scenarios like **use-after-free** or **data races**.
## 2. Dereferencing Smart Pointers
Smart pointers in Rust, such as `Box<T>`, `Rc<T>`, and `RefCell<T>`, manage ownership of data and provide controlled dereferencing through traits like `Deref` and `DerefMut`.
### Using `Box<T>`
A `Box<T>` is a smart pointer for heap-allocated data. Dereferencing a `Box` gives you access to the underlying value.
```rust
fn main() {
    let boxed = Box::new(5); // Box allocates the value on the heap

    println!("Value through Box: {}", *boxed); // Dereferencing the Box
}
```
### Using `Rc<T>`
`Rc<T>` is a reference-counted smart pointer, allowing shared ownership. You can dereference an `Rc` like a regular pointer.
```rust
use std::rc::Rc;

fn main() {
    let rc = Rc::new(10);

    println!("Value through Rc: {}", *rc); // Dereferencing Rc
}
```
### `Deref` and `DerefMut` Traits`
Rust uses the `Deref` and `DerefMut` traits to define custom dereferencing behavior for smart pointers.
#### Rules:
1. **Immutable Dereferencing (`Deref`)**:
    - Used when dereferencing immutable smart pointers (`Box`, `Rc`).
    - Example:
```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

fn main() {
    let my_box = MyBox(42);
    println!("Value through MyBox: {}", *my_box); // Dereferencing via Deref
}
```
**Mutable Dereferencing (`DerefMut`)**:
- Used when dereferencing mutable smart pointers (`RefCell` or `Box` in some cases).
- Example:
```rust
use std::ops::DerefMut;

struct MyBox<T>(T);

impl<T> DerefMut for MyBox<T> {
    fn deref_mut(&mut self) -> &mut Self::Target {
        &mut self.0
    }
}

fn main() {
    let mut my_box = MyBox(42);
    *my_box += 1; // Mutable dereferencing via DerefMut
    println!("Updated value: {}", *my_box);
}
```
## 3. Dereferencing Raw Pointers
Raw pointers (`*const T` and `*mut T`) are used for low-level memory access. Dereferencing them is unsafe because Rust cannot guarantee their validity.
#### Example:
```rust
fn main() {
    let x = 42;
    let raw_ptr = &x as *const i32; // Raw immutable pointer

    unsafe {
        println!("Value through raw pointer: {}", *raw_ptr); // Dereferencing raw pointer
    }
}
```
#### Rules:
- Dereferencing raw pointers requires an `unsafe` block.
- Use with extreme care to avoid invalid memory access or undefined behavior.
## 4. Deref Coercion
**Deref coercion** is a convenience feature in Rust that automatically converts (or coerces) references to a type when the target type implements the `Deref` trait. It allows you to call methods or access fields of the target type as if the reference were already dereferenced, reducing the need for explicit dereferencing (`*`).

This feature is commonly used to make smart pointers like `Box`, `Rc`, or `Arc` behave like regular references, enabling ergonomic access to the data they manage.
### Example:
```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let name = MyBox(String::from("Rust"));

    hello(&name); // Deref coercion: MyBox<String> -> &String -> &str
}
```
### How Deref Coercion Works
If a type implements the `Deref` trait, Rust automatically inserts calls to the `deref` method when it sees that a reference to a type needs to be converted to a reference of another type.
**Signature of the `Deref` Trait**:
```rust
use std::ops::Deref;

trait Deref {
    type Target: ?Sized; // The type being pointed to (can be unsized)
    fn deref(&self) -> &Self::Target;
}
```
The `deref` method is used to return a reference to the inner value (`&Self::Target`).
#### Example of Deref Coercion:
```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

fn main() {
    let x = MyBox(5);
    let y = *x; // Works because `Deref` is implemented
    println!("{}", y); // Prints 5
}
```
Here, `MyBox` behaves like a reference to the inner type (`T`) because of the `Deref` implementation.
### Common Use Cases
1. **Smart Pointers**: Rust's built-in smart pointers like `Box<T>`, `Rc<T>`, and `Arc<T>` implement `Deref`. This allows you to treat smart pointers as regular references.
Example:
```rust
use std::rc::Rc;

let s = Rc::new(String::from("Hello, Rust!"));
println!("{}", s.len()); // Automatically dereferences Rc<String> to String
```
The `Deref` trait allows the compiler to interpret `s.len()` as:
```rust
println!("{}", (*s).len());
```
2. **String Slices (`&str`) from `String`**: The `String` type implements `Deref` with `Target = str`. This means you can use a `String` where a `&str` is expected.
Example:
```rust
fn greet(name: &str) {
    println!("Hello, {}!", name);
}

let name = String::from("Alice");
greet(&name); // Deref coercion: &String -> &str
```
Without deref coercion, you'd need to write:
```rust
greet(&name[..]); // Explicitly taking a slice
```
3. **Trait Object Coercion (`dyn Trait`)**: Deref coercion is also used when converting between references to trait objects.
Example:
```rust
trait Drawable {
    fn draw(&self);
}

struct Circle;

impl Drawable for Circle {
    fn draw(&self) {
        println!("Drawing a circle.");
    }
}

fn display(item: &dyn Drawable) {
    item.draw();
}

let circle = Circle;
display(&circle); // Deref coercion to &dyn Drawable
```
### Rules for Deref Coercion
1. **Chaining**: Deref coercion can apply recursively. If `T` dereferences to `U`, and `U` dereferences to `V`, the compiler can perform a chained coercion from `T` to `V`.
Example:
```rust
let s = Rc::new(Box::new(String::from("Hello")));
println!("{}", s.len()); // Rc<Box<String>> -> Box<String> -> String
```
2. **Shared and Mutable References**:
	- **Shared references (`&T`)**: The `deref` method is called.
	- **Mutable references (`&mut T`)**: The `deref_mut` method (from the `DerefMut` trait) is called.
Example:
```rust
use std::ops::{Deref, DerefMut};

struct MyBox<T>(T);

impl<T> Deref for MyBox<T> {
    type Target = T;
    fn deref(&self) -> &T {
        &self.0
    }
}

impl<T> DerefMut for MyBox<T> {
    fn deref_mut(&mut self) -> &mut T {
        &mut self.0
    }
}

fn modify(value: &mut i32) {
    *value += 1;
}

let mut x = MyBox(5);
modify(&mut x); // Deref coercion to &mut i32
println!("{}", *x); // Prints 6
```
3. **No Deref Coercion for Raw Pointers**: Deref coercion does not apply to raw pointers (`*const T` or `*mut T`) because raw pointers are not safe to dereference directly.
	- You must explicitly use `unsafe` to dereference raw pointers.
Example:
```rust
let x: i32 = 10;
let raw: *const i32 = &x;

// println!("{}", *raw); // Error: cannot dereference raw pointers safely
unsafe {
    println!("{}", *raw); // Explicit unsafe block
}
```
### Limitations and Notes
1. **Explicit Deref Needed in Some Cases**: While deref coercion is powerful, it might not apply in every situation. For instance, when method resolution is ambiguous, you might need to explicitly dereference.
Example:
```rust
struct A;
struct B;

impl A {
    fn method(&self) {
        println!("A's method");
    }
}

impl B {
    fn method(&self) {
        println!("B's method");
    }
}

impl std::ops::Deref for B {
    type Target = A;

    fn deref(&self) -> &A {
        &A
    }
}

let b = B;

// b.method(); // Error: ambiguous method resolution
(*b).method(); // Explicit dereference to call A's method
```
2. **Cannot Coerce to Arbitrary Types**: Deref coercion only works for types that implement `Deref` (or `DerefMut`). You cannot coerce arbitrary types into each other without explicit conversions.
### Summary
- **Deref coercion** simplifies the syntax when working with references, smart pointers, and trait objects.
- It relies on the `Deref` (or `DerefMut`) trait to allow implicit dereferencing.
- Common use cases include:
    - Smart pointers like `Box`, `Rc`, and `Arc`.
    - Converting `String` to `&str`.
    - Trait object coercion.
- For more advanced use cases, explicit dereferencing (`*`) might still be required.
---
## 5. Safety Rules
### Immutable Dereferencing
- Always safe because the data cannot be modified.
- Multiple immutable de-references are allowed.
### Mutable Dereferencing
- Safe only if no other references (mutable or immutable) exist to the same value.
- Rust’s borrow checker enforces this at compile time.
### Raw Pointers
- Dereferencing requires `unsafe` because Rust cannot validate memory safety.
- Use when interacting with low-level C APIs or doing manual memory management.
---
## Summary of Rules

| Reference Type | Dereferencing Allowed | Safety | Key Rule                                   |
| -------------- | --------------------- | ------ | ------------------------------------------ |
| `&T`           | Read                  | Safe   | Multiple immutable borrows allowed.        |
| `&mut T`       | Read and Write        | Safe   | Only one mutable borrow allowed at a time. |
| `Box<T>`       | Read and Write        | Safe   | Managed ownership with heap allocation.    |
| `Rc<T>`        | Read                  | Safe   | Shared ownership with reference counting.  |
## The 'ref' keyword
Sometimes you may be want to use some portion of a tuple or a struct as a reference while other parts as owned values. For instance, consider the code below
```rust
fn main() {
     let tuple = (String::from("Hello"), 42);
     let (s, num) = tuple;
}
```

We would like 's' to use a reference to string, while `num` as owned value. If we use the below code:
```rust
 let (s, num) = &tuple;
```
it will make both `s` and `num` as references. To achieve the desired behaviour, we can use the ref keyword. The updated statement will look like
```rust
let (ref s, num) = tuple;
```
This will make the type of s as a &String while `num` will remain as owned value with a type of i32. Although specific use cases of this is rare, this is just good to know.