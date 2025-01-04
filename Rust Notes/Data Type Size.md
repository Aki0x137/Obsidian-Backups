![[rust_collection_types.png]]
# Sized and Unsized Types in Rust
In Rust, types are categorized as **sized** or **unsized** based on whether their size is known at compile time.
# 1. Sized Types
- **Definition**: These are types whose size is known at compile time.
- **Examples**: Primitive types (`i32`, `f64`), arrays with fixed sizes (`[i32; 5]`), and most structs or enums.
Rust assumes that types are **sized by default**. This assumption is expressed through the **`Sized` trait**, which is automatically implemented for all types whose size is known at compile time.
**Example**:
```rust
let x: i32 = 42;  // i32 is a sized type
let array: [i32; 4] = [1, 2, 3, 4]; // Fixed-size arrays are sized
```
## Key Points:
- **Stack Allocation**: Sized types are typically stored directly on the stack.
- **Default Behavior**: Most types are `Sized`.
# 2. Unsized Types
- **Definition**: These are types whose size is **not known** at compile time.
- **Examples**:
    - **Slices** (`[T]`): Dynamic-length arrays.
    - **Trait Objects** (`dyn Trait`): Dynamically dispatched trait implementations.
    - **Str** (`str`): A dynamically-sized string slice.
## Characteristics:
- Cannot be used directly; they must always be behind a pointer type like `&`, `Box`, or `Rc`.
- Rust uses **fat pointers** to handle them.
**Example**:
```rust
let s: &str = "Hello, Rust!"; // &str is an unsized type behind a reference
let slice: &[i32] = &[1, 2, 3]; // &[i32] is a slice, an unsized type
```

# Fat and Thin Pointers
To manage **unsized types**, Rust uses two kinds of pointers:
## 1. Thin Pointers
- Point directly to the memory address of the value.
- Used for **sized types**, where the size is known at compile time.
**Example**:
```rust
let x = 42;
let ptr: &i32 = &x; // Thin pointer, points directly to x
```
## 2. Fat Pointers
- Used for **unsized types**.
- Contain:
    1. A pointer to the data.
    2. Metadata that describes the size or layout of the type at runtime.

### Fat Pointer Examples:
- **Slices** (`[T]`):
    - Metadata: Length of the slice.
    - Pointer: Address of the first element.
    - Example:
```rust
let slice: &[i32] = &[1, 2, 3];
```
A fat pointer to this slice includes:
	- The memory address of the first element (`&slice[0]`).
	- The length (`3`).
- **Trait Objects** (`dyn Trait`):
	- Metadata: A pointer to a **vtable** (virtual method table) that describes the methods of the trait implementation.
	- Pointer: Address of the concrete type implementing the trait.
	- Example:
```rust
trait MyTrait {
    fn my_method(&self);
}

struct MyStruct;

impl MyTrait for MyStruct {
    fn my_method(&self) {
        println!("Hello from MyStruct");
    }
}

let obj: &dyn MyTrait = &MyStruct; // Fat pointer: points to MyStruct and its vtable
```
## Sized Trait
- **Default Trait**: All types are assumed to be `Sized` unless explicitly marked otherwise.
- **Unsized Example**: To allow a type to work with unsized types, you can use the `?Sized` bound. \[IMPORTANT\]
- Example:
```rust
struct Wrapper<T: ?Sized> {
    value: T,
}
```
Here:
- The `?Sized` bound indicates that `T` may be either sized or unsized.
- Without this, `T` would default to being `Sized`.
## Thin vs Fat Pointer Summary

|Feature|Thin Pointer|Fat Pointer|
|---|---|---|
|**Usage**|Sized types|Unsized types|
|**Size**|One word (address)|Two words (address + metadata)|
|**Metadata**|None|Length (for slices) / vtable (for traits)|
|**Examples**|`&i32`, `Box<u32>`|`&[T]`, `&str`, `&dyn Trait`|
# Why Use Unsized Types?
1. **Dynamic Data Structures**:
    - Slices (`[T]`) allow for arrays of dynamic length.
    - Example: Processing dynamic input arrays.
2. **Trait Objects for Polymorphism**:
    - Enable dynamic dispatch via `dyn Trait`.
    - Example: Implementing polymorphic behavior for types implementing a common trait.
3. **Efficient Memory Usage**:
    - Unsized types like slices or `str` prevent copying large amounts of data.
# Use Cases for Sized and Unsized Types
## Sized Types:
- Use whenever you need predictable memory allocation (e.g., `i32`, `f64`, fixed-size arrays, structs).
## Unsized Types:
- Use slices (`[T]`) for arrays with unknown size at compile time.
- Use `dyn Trait` for polymorphism when working with trait objects.
- Use `str` when handling dynamically sized string data.
# References to Unsized Types in Rust
In Rust, **references to unsized types** are special because unsized types themselves cannot exist without some form of metadata that describes their size or layout. Therefore, these references require additional handling through **fat pointers**.
## How References to Unsized Types Work
A reference to an unsized type consists of:
1. A **pointer to the actual data** in memory.
2. **Metadata** that provides runtime information about the type.
Rust uses **fat pointers** to store both the pointer and metadata together.
### Metadata for Unsized Types:
- **Slices** (`[T]`): The metadata is the length of the slice.
- **String Slices** (`str`): The metadata is the length of the string.
- **Trait Objects** (`dyn Trait`): The metadata is a pointer to the **vtable**, which stores function pointers and information about the trait.
## 1. References to Slices (`[T]`)
A slice represents a dynamically sized view of an array or other collection.
**Example**:
```rust
let array: [i32; 4] = [1, 2, 3, 4];
let slice: &[i32] = &array; // Reference to the slice
```
The reference `&[i32]` is a fat pointer that:
- Points to the first element of the array.
- Contains the length of the slice (`4` in this case).
**Behind the Scenes**:
```rust
(slice) -> { pointer: &array[0], metadata: 4 (length) }
```
## 2. References to Strings (`str`)
The `str` type is a dynamically sized string slice. It is always used behind a reference like `&str`.
**Example**:
```rust
let s = String::from("Hello, Rust!");
let slice: &str = &s;
```
- The reference `&str`:
    - Points to the first byte of the string.
    - Contains the length of the string (`12` in this case).
**Behind the Scenes**:
```rust
(slice) -> { pointer: &s[0], metadata: 12 (length) }
```
## 3. References to Trait Objects (`dyn Trait`)
Trait objects are unsized because the concrete type implementing the trait is not known at compile time. A reference to a trait object (`&dyn Trait`) is stored as a fat pointer.
**Example**:
```rust
trait MyTrait {
    fn say_hello(&self);
}

struct MyStruct;

impl MyTrait for MyStruct {
    fn say_hello(&self) {
        println!("Hello from MyStruct!");
    }
}

let obj: &dyn MyTrait = &MyStruct;
```
- The reference `&dyn MyTrait` consists of:
    1. A pointer to the concrete value (`MyStruct` in this case).
    2. A pointer to the vtable for the trait (`MyTrait`).
**Behind the scenes**:
```rust
(obj) -> { pointer: &MyStruct, metadata: &vtable }
```
## Operations on References to Unsized Types
### Accessing Data
You can access data behind the reference just as you would with a sized type, but Rust uses the metadata to ensure safety.
### Slicing Strings
```rust
let s = String::from("Hello, Rust!");
let slice: &str = &s[0..5]; // "Hello"
println!("{}", slice);
```
### Dynamic Dispatch with Trait Objects
When calling a method on a `dyn Trait`, Rust uses the vtable to resolve which method to call at runtime.
```rust
obj.say_hello(); // Uses the vtable to call MyStruct's implementation
```
## Common Pointer Types for Unsized Types \[IMPORTANT\]
Unsized types must always be placed behind some kind of pointer. Common options include:
1. **Shared References (`&`)**:
    - Most common for read-only access.
    - Example: `&[T]`, `&str`, `&dyn Trait`.
2. **Mutable References (`&mut`)**:
    - Used when you need to mutate the underlying value.
    - Example: `&mut [T]`, `&mut dyn Trait`.
3. **Heap-Allocated Pointers (`Box`)**:
    - Used to allocate unsized types on the heap.
    - Example: `Box<[T]>`, `Box<dyn Trait>`.
4. **Reference-Counting (`Rc`/`Arc`)**:
    - Used for shared ownership of unsized types.
    - Example: `Rc<dyn Trait>`, `Arc<[T]>`.

## **Practical Use Cases**
1. **Dynamically Sizing Arrays with Slices**
    - Slices are useful when you want to handle data collections without knowing their size at compile time.
    - Example: Functions that accept any array size:
```rust
fn print_slice(slice: &[i32]) {
    for item in slice {
        println!("{}", item);
    }
}
```
**2. Polymorphism with Trait Objects**
- Trait objects allow you to use dynamic dispatch for heterogeneous types implementing the same trait.
- Example: Storing different types in a collection:
```rust
let items: Vec<Box<dyn MyTrait>> = vec![Box::new(MyStruct)];
```
**3. String Manipulation**
- Use `&str` to work with string slices without copying data.
## **Summary**
- References to unsized types (`[T]`, `str`, `dyn Trait`) require **fat pointers**.
- Fat pointers combine a data pointer and metadata to safely manage dynamic sizes or polymorphism.
- You can use shared references, mutable references, heap pointers (`Box`), or reference counting (`Rc`, `Arc`) to handle unsized types.
- They are crucial for working with dynamically sized data (e.g., slices, trait objects) or implementing polymorphism.
# `?Sized`
[[Generics and Traits#1. `?Sized`]]
# Structs and unsized fields
In Rust, **structs have specific limitations when dealing with unsized fields**, primarily due to how memory layout and size computation work. Since Rust relies heavily on knowing the size of types at compile time for memory allocation, unsized fields introduce unique constraints.
## Key Rules and Limitations for Struct Fields with Unsized Types

### 1. Only One unsized field
- A struct can only have one unsized field.
- Structs cannot contain more than one unsized field. Since only the last field of a struct can be unsized, this naturally limits the number of unsized fields to one.
```rust
struct Invalid<T: ?Sized, U: ?Sized> {
    field1: T, // Error: Only one unsized field is allowed.
    field2: U,
}
```
### 2. Unsized Fields Must Be the Last Field
- It must be the **last field** of the struct. This ensures that the compiler can compute the offset of all other fields in the struct.
**Why?**
- Structs in Rust have a fixed size at compile time, except for their last field if it's unsized. The size of earlier fields must be known to calculate offsets for accessing those fields.
```rust
struct Valid<T: ?Sized> {
    fixed: i32,
    dynamic: T, // This is valid because it's the last field.
}

struct Invalid<T: ?Sized> {
    dynamic: T, // Error: Unsized field cannot be in the middle.
    fixed: i32,
}
```
### 3. Unsized Fields Must Be Behind a Pointer
- Unsized types cannot exist directly in a struct unless they are **behind a pointer type** like `Box`, `Rc`, or `Arc`. This is because unsized types themselves lack a concrete size at compile time.
**Why?**
- Without a pointer, the memory layout and size of the struct cannot be determined.
```rust
struct Valid<T: ?Sized> {
    fixed: i32,
    dynamic: Box<T>, // Valid because T is behind a pointer.
}

struct Invalid<T: ?Sized> {
    dynamic: T, // Error: Unsized field must be behind a pointer.
}
```
### 4. Structs with Unsized Fields Are Dynamically Sized
- If a struct has an unsized field, the struct itself becomes **dynamically sized** (`!Sized`). Such structs cannot be used directly but must also be placed behind a pointer, such as `&`, `Box`, or `Rc`.
**Example**:
```rust
struct Container<T: ?Sized> {
    value: T,
}

// Error: Cannot create a dynamically sized struct directly.
let container = Container { value: "hello" };

// Correct: Place it behind a pointer like Box.
let container = Box::new(Container { value: "hello" });
```
## Memory Layout of Structs with Unsized Fields
For structs with unsized fields, the compiler stores metadata (e.g., the length of a slice or the vtable for a trait object) alongside the pointer to manage the unsized field.
**Example**: Struct with a Slice
```rust
struct SliceHolder<T: ?Sized> {
    length: usize,
    data: [T],
}

// Usage
let data: &[u8] = &[1, 2, 3];
let holder = SliceHolder {
    length: data.len(),
    data: *data, // This is invalid without a pointer.
};
```
To fix this, you need to place the unsized field behind a pointer:
```rust
struct SliceHolder<T: ?Sized> {
    length: usize,
    data: Box<[T]>, // Unsized slice field behind a pointer.
}

let data: &[u8] = &[1, 2, 3];
let holder = SliceHolder {
    length: data.len(),
    data: data.into(), // Convert to Box<[T]>
};
```
### Special Cases with Trait Objects
#### Structs with `dyn Trait` Fields
- Trait objects (`dyn Trait`) are inherently unsized, so they must also be the last field and placed behind a pointer:
```rust
struct TraitHolder {
    name: String,
    behavior: Box<dyn std::fmt::Debug>, // `dyn Debug` is unsized, so it must be behind a pointer.
}
```
## Practical Examples
### Example 1: Struct with a Dynamically Sized Field
```rust
struct CustomString {
    length: usize,
    content: str, // Error: `str` must be the last field and unsized.
}

// Correct:
struct CustomString {
    length: usize,
    content: Box<str>, // Place `str` behind a pointer.
}

let my_string = CustomString {
    length: 5,
    content: "hello".into(), // Convert to Box<str>.
};
```
### Example 2: Unsized Types in Generic Structs
```rust
struct Container<T: ?Sized> {
    name: String,
    value: Box<T>, // `T` is unsized but placed behind a pointer.
}

let my_container: Container<dyn std::fmt::Debug> = Container {
    name: "example".to_string(),
    value: Box::new(42), // Box<dyn Debug>
};
```
### Example 3: Combining Unsized Types and Slices
```rust
struct SliceWrapper<T: ?Sized> {
    length: usize,
    data: [T], // Error: `[T]` must be behind a pointer.
}

// Correct version:
struct SliceWrapper<T: ?Sized> {
    length: usize,
    data: Box<[T]>, // Dynamically sized `[T]` behind a pointer.
}

let numbers = vec![1, 2, 3];
let wrapper = SliceWrapper {
    length: numbers.len(),
    data: numbers.into_boxed_slice(),
};
```
## Multiple dynamic size fields
> [!tip] If you need more than one dynamically sized field in a struct in Rust, you cannot directly store multiple unsized fields (e.g., slices, `dyn Trait`, or `str`) because Rust requires that only the last field of a struct be unsized. However, you can work around this limitation by using **pointers** (e.g., `Box`, `Rc`, `Arc`, or references) to store each unsized field. This approach ensures that the struct itself remains `Sized`, and only the pointed-to data is dynamically sized.
### 1. Use Pointers for Each Dynamically Sized Field
Instead of directly embedding unsized fields in the struct, place them behind smart pointers (e.g., `Box`, `Rc`, or `Arc`) or references.
```rust
struct MultiDynamic {
    name: Box<str>,          // Dynamically sized string slice
    data: Box<[u8]>,         // Dynamically sized slice of bytes
    behavior: Box<dyn Fn()>, // Trait object
}

fn main() {
    let dynamic_struct = MultiDynamic {
        name: "Hello".into(),             // Box<str>
        data: vec![1, 2, 3].into_boxed_slice(), // Box<[u8]>
        behavior: Box::new(|| println!("Hello, World!")), // Box<dyn Fn()>
    };

    println!("{}", dynamic_struct.name);
    println!("{:?}", dynamic_struct.data);
    (dynamic_struct.behavior)();
}
```
This approach ensures that the struct is `Sized` because all dynamically sized fields are stored behind pointers.
### 2. Nested Structs
If you want to group related unsized fields together, you can use **nested structs** where each struct has a single unsized field. Each nested struct becomes dynamically sized, but the outer struct remains `Sized` as long as it uses pointers.
**Example**:
```rust
struct NameField {
    name: str, // Unsized field
}

struct DataField {
    data: [u8], // Unsized field
}

struct MultiDynamic {
    name: Box<NameField>,
    data: Box<DataField>,
}

fn main() {
    let name = Box::new(NameField { name: *"Hello" });
    let data = Box::new(DataField { data: *[1, 2, 3] });

    let dynamic_struct = MultiDynamic { name, data };

    println!("{}", dynamic_struct.name.name);
    println!("{:?}", dynamic_struct.data.data);
}
```
By nesting the unsized fields inside their own structs, you can manage multiple unsized fields while keeping the top-level struct usable.
# Unsized coercion

**Unsized coercion** refers to the automatic conversion of a type with a known size at compile time (a "sized type") to a type without a known size at compile time (an "unsized type"). This feature is most commonly used in Rust with dynamically sized types (**DSTs**) like slices (`[T]`), string slices (`str`), and trait objects (`dyn Trait`).
> [!info]
> It is related to deref coercion [[Ownership & Borrowing#4. Deref Coercion]]
## What is Unsized Coercion?
Unsized coercion allows Rust to automatically convert a sized type into a **reference or pointer to an unsized type**. This conversion is **lossless**, as the unsized type is always accompanied by metadata (e.g., the length of a slice or the vtable for a trait object) to ensure it can be used safely.
### Key Properties
1. Unsized coercion only works when the source type "dynamically dereferences" into the target type.
2. The result of coercion must be a pointer or reference type (`&`, `&mut`, `Box`, `Rc`, etc.).
3. The coercion is implicit and happens automatically in certain contexts, like function arguments or when assigning values.
## Common Cases of Unsized Coercion
1. **From Arrays to Slices (`[T; N]` → `[T]`)**
    - A fixed-size array (`[T; N]`) can be coerced into a slice (`[T]`) because slices are a dynamically sized view into a contiguous sequence of elements.
```rust
fn print_slice(slice: &[i32]) {
    for elem in slice {
        println!("{}", elem);
    }
}

fn main() {
    let arr = [1, 2, 3];
    print_slice(&arr); // Coerces &[i32; 3] into &[i32]
}
```
- The compiler automatically converts `&[i32; 3]` into `&[i32]`, carrying the length of the array as metadata.
2. **From `String` to `str`**
    - The `String` type is a heap-allocated, grow-able string, whereas `str` is an unsized string slice. You can coerce `&String` to `&str`.
    - The coercion works because `String` implements `Deref<Target = str>`.
```rust
fn greet(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let name = String::from("Alice");
    greet(&name); // Coerces &String into &str
}
```
3. **From Structs with Unsized Fields**
    - A struct with its last field as an unsized type can coerce into a reference to an unsized version of the struct.
    - Here, the coercion happens because the struct's last field is dynamically sized.
```rust
struct Foo<T: ?Sized> {
    value: T,
}

fn main() {
    let foo: &Foo<[i32]> = &Foo { value: [1, 2, 3] };
    println!("{:?}", foo.value);
}
```
4. **From Concrete Types to Trait Objects (`T` → `dyn Trait`)**
    - A type implementing a trait can be coerced into a reference to the corresponding trait object (`dyn Trait`).
    - This coercion works because `dyn Trait` includes a vtable to enable dynamic dispatch.
```rust
trait Speak {
    fn speak(&self);
}

struct Dog;

impl Speak for Dog {
    fn speak(&self) {
        println!("Woof!");
    }
}

fn make_speak(speaker: &dyn Speak) {
    speaker.speak();
}

fn main() {
    let dog = Dog;
    make_speak(&dog); // Coerces &Dog into &dyn Speak
}
```
## When Does Unsized Coercion Happen?
1. **Function Arguments** When a function expects a reference to an unsized type, unsized coercion is applied automatically.
```rust
fn process(slice: &[i32]) { /* ... */ }

let arr = [1, 2, 3];
process(&arr); // Coerces &[i32; 3] into &[i32]
```
2. **Assignments** When assigning a reference or pointer to an unsized type.
```rust
let s: &str = &String::from("hello"); // Coerces &String into &str
```
3. **Trait Objects** When a reference or pointer to a concrete type is assigned to a variable of type `dyn Trait`.
```rust
let obj: &dyn Speak = &Dog; // Coerces &Dog into &dyn Speak
```
4. **Return Values** When a function returns a reference or pointer to an unsized type.
```rust
fn get_slice(arr: &[i32; 3]) -> &[i32] {
    arr // Coerces &[i32; 3] into &[i32]
}
```
## Constraints
1. **Works Only for Reference or Pointer Types** Unsized coercion cannot convert values directly—only references or pointers to them. For example:
2. **The Target Type Must Be a DST** The target type must be unsized (e.g., `str`, `[T]`, `dyn Trait`).
3. **Requires Metadata** The unsized type must have metadata (e.g., length for slices, vtables for trait objects). If no metadata is available, coercion is not possible.
### Why is Unsized Coercion Useful?
- **Ergonomics**: Reduces the need for explicit conversions between sized and unsized types.
- **Compatibility**: Allows functions and data structures to work seamlessly with dynamically sized data (e.g., slices, trait objects).
- **Dynamic Behavior**: Enables dynamic dispatch with trait objects (`dyn Trait`), allowing polymorphism at runtime.
### Key Takeaways
- **Unsized coercion** allows conversion from a sized type to a reference or pointer to an unsized type.
- Common cases include coercing arrays to slices (`[T; N] → [T]`), strings to string slices (`String → str`), and concrete types to trait objects (`T → dyn Trait`).
- It requires metadata, which is carried along with the reference or pointer.
- It works implicitly in contexts like function arguments, assignments, and trait object usage.