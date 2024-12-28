# Generics
Generics in Rust allow to write flexible, reusable, and type-safe code by enabling functions, structs, enums, and traits to work with multiple types while maintaining strict type safety. This is achieved using **type parameters**.
## **1. Syntax**
The generic type parameter is specified using angle brackets `<T>`.
**Generic Function**:
```rust
fn print_value<T>(value: T) {
    println!("{:?}", value);
}
```
**Generic Struct**:
```rust
struct Point<T, U> {
    x: T,
    y: U,
}
```
**Generic Enum**:
```rust
enum Option<T> {
    Some(T),
    None,
}
```
## **2. How It Works**
Generics are monomorphized during compilation, meaning Rust generates specific implementations of the code for each type that is used. This ensures zero runtime overhead.
```rust
fn square<T: std::ops::Mul<Output = T>>(x: T) -> T {
    x * x
}

fn main() {
    println!("{}", square(4));      // Works with integers
    println!("{}", square(3.14));  // Works with floats
}
```
## **3. Constraints (Traits)**
You can constrain generic types to ensure they implement certain traits. For example:
**With a Trait Bound**:
```rust
fn print_sum<T: std::ops::Add<Output = T>>(a: T, b: T) {
    println!("{}", a + b);
}
```
## **4. Real-World Use Cases**
- **Collections**: The `Vec<T>` and `HashMap<K, V>` collections in Rust are implemented using generics.
- **Option and Result Types**: Both `Option<T>` and `Result<T, E>` are generic enums.
- **Custom Data Structures**: Generic types are ideal for creating reusable structs like `Point<T>` or generic trees.
## **5. Key Benefits of Generics**
- **Code Re-usability**: Avoid writing duplicate code for multiple types.
- **Type Safety**: Compile-time type checking ensures correctness.
- **Zero Runtime Cost**: Generics are monomorphized into specific implementations at compile time.
# Traits
In Rust, **traits** are a powerful mechanism for defining shared behavior across types. Traits are like interfaces in other languages—they specify a set of methods that a type must implement to satisfy the trait.
## **1. What Are Traits?**
A trait is a collection of method signatures that define behavior. If a type implements a trait, it guarantees the implementation of those methods.
**Example: Defining a Trait**
```rust
trait Speak {
    fn speak(&self) -> String; // A method signature
}
```
**Implementing a Trait**:
```rust
struct Dog;

impl Speak for Dog {
    fn speak(&self) -> String {
        "Woof!".to_string()
    }
}

fn main() {
    let dog = Dog;
    println!("{}", dog.speak());
}
```
In this example:
- The `Speak` trait defines a behavior.
- The `Dog` type implements the `Speak` trait.
## **2. Traits with Default Implementations**
Traits can include default implementations for methods. Types implementing the trait can override these defaults.
```rust
trait Greet {
    fn greet(&self) {
        println!("Hello!");
    }
}

struct Person;

impl Greet for Person {}

fn main() {
    let person = Person;
    person.greet(); // Prints: Hello!
}
```
Here:
- `Greet` provides a default `greet` method.
- The `Person` type does not override it but inherits the default behavior.
## **3. Traits for Operator Overloading**
Traits like `Add`, `Sub`, and others allow for operator overloading in Rust.
```rust
use std::ops::Add;

struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    let p3 = p1 + p2;

    println!("Point: ({}, {})", p3.x, p3.y); // Point: (4, 6)
}
```
## **4. Blanket Implementations**
Rust allows you to implement traits for all types that satisfy certain bounds.
#### Example: Implementing a Trait for All Types
```rust
trait Greet {
    fn greet(&self) {
        println!("Hello!");
    }
}

impl<T> Greet for T {}

fn main() {
    42.greet(); // Works because Greet is implemented for all types
}
```
## **5. Trait Objects (Dynamic Dispatch)**
If you need to work with multiple types implementing the same trait, you can use **trait objects**.
#### Example: Dynamic Dispatch
```rust
trait Speak {
    fn speak(&self);
}

struct Dog;
struct Cat;

impl Speak for Dog {
    fn speak(&self) {
        println!("Woof!");
    }
}

impl Speak for Cat {
    fn speak(&self) {
        println!("Meow!");
    }
}

fn make_speak(animal: &dyn Speak) {
    animal.speak();
}

fn main() {
    let dog = Dog;
    let cat = Cat;

    make_speak(&dog); // Woof!
    make_speak(&cat); // Meow!
}
```
Here:
- `&dyn Speak` is a **trait object**, enabling dynamic dispatch.
## **6. Auto Traits**
Certain traits, like `Send` and `Sync`, are automatically implemented for types where applicable.
**Example**:
```rust
fn requires_send<T: Send>(item: T) {
    // Only types that implement `Send` can be passed
}
```
## **7. Common Traits in Rust**

| Trait        | Purpose                                          |
| ------------ | ------------------------------------------------ |
| `Clone`      | Allows duplicating objects.                      |
| `Copy`       | Allows bitwise copy for types.                   |
| `Debug`      | Allows formatting for debugging.                 |
| `PartialEq`  | Enables comparison using `==`.                   |
| `PartialOrd` | Enables ordering using `<`, `>`.                 |
| `Iterator`   | Provides iteration behavior.                     |
| `Drop`       | Defines behavior when a value goes out of scope. |
# Trait Bounds
Trait bounds allow you to use traits as constraints on generic types, ensuring that types used in generic code have specific behavior.
## **1. Trait Bounds**
Trait bounds are used to constrain generic types, ensuring they implement specific traits.
**Basic Syntax**:
```rust
fn print_speak<T: Speak>(item: T) {
    println!("{}", item.speak());
}
```
**Example with Trait Bound**:
```rust
trait Speak {
    fn speak(&self) -> String;
}

struct Dog;

impl Speak for Dog {
    fn speak(&self) -> String {
        "Woof!".to_string()
    }
}

struct Cat;

impl Speak for Cat {
    fn speak(&self) -> String {
        "Meow!".to_string()
    }
}

fn animal_speak<T: Speak>(animal: T) {
    println!("{}", animal.speak());
}

fn main() {
    let dog = Dog;
    let cat = Cat;

    animal_speak(dog); // Woof!
    animal_speak(cat); // Meow!
}
```
## **2. Advanced Syntax for Trait Bounds**
#### Using the `where` Clause
For complex bounds, use `where` for better readability.
```rust
fn print_details<T>(item: T)
where
    T: std::fmt::Debug + Clone,
{
    println!("{:?}", item.clone());
}
```
#### Multiple Trait Bounds
```rust
fn process<T: Trait1 + Trait2>(item: T) {
    // Item must implement both Trait1 and Trait2
}
```
# **Key Points**
1. Traits are like interfaces that define shared behavior.
2. Trait bounds constrain generic types to ensure required behavior.
3. Traits can have default method implementations.
4. Use `pub trait` for public APIs.
5. Traits support static dispatch (compile-time) and dynamic dispatch (runtime).