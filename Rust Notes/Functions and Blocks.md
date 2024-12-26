# Functions

- The last expression in a function is its return value (no `return` keyword needed).
- Function Name must be snake_case by convention.

## **Default Parameters**:
- Rust doesn’t have default parameters. Use optional types (`Option`) or provide defaults in the caller.
```rust
fn greet(name: Option<&str>) {
    let name = name.unwrap_or("Guest");
    println!("Hello, {}!", name);
}
```
## Closures
Closures are inline, anonymous functions that can capture their environment.
```rust
let add = |a, b| a + b;
println!("{}", add(2, 3)); // 5
```
## Ownership and Lifetimes in Functions
Rust’s ownership system ensures memory safety. Parameters can be:
- Moved
- Borrowed (immutable or mutable)
- Cloned
```rust
fn takes_ownership(s: String) {
    println!("{}", s); // Ownership moves here
}

fn borrows_value(s: &str) {
    println!("{}", s); // Borrowed, no ownership transfer
}

fn main() {
    let s = String::from("Rust");
    takes_ownership(s); // `s` is now invalid

    let t = String::from("Borrowing");
    borrows_value(&t); // `t` is still valid
}
```
# **Blocks
A block is a group of statements enclosed in `{}`. Blocks are used in various places, such as functions, control structures, and standalone expressions.
```rust
fn main() {
    let x = {
        let a = 5;
        let b = 10;
        a + b // The block evaluates to `15`
    };
    
	let y = {
		let a = 5;
		let b = 10;
		a + b;
	}; // The block evaluates to () - empty tuple


    println!("{}", x); // 15
}
```
- The last expression in a block (without a semicolon) determines its value.
- Adding a semicolon (`;`) converts the block into a statement, which does not return a value.
## Nested Blocks
Blocks can be nested to create local scopes.
```rust
fn main() {
    let x = 10;

    let result = {
        let x = 20; // Shadowing the outer `x`
        x + 5 // 25
    };

    println!("{}", x); // 10
    println!("{}", result); // 25
}
```
## Control Flow and Blocks
Blocks are used in control structures like `if`, `match`, and loops.
### `if` with Blocks
```rust
fn main() {
    let condition = true;

    let value = if condition {
        42
    } else {
        0
    };

    println!("{}", value); // 42
}
```
### `match` with Blocks
```rust
fn main() {
    let number = 3;

    let result = match number {
        1 => "One",
        2 => "Two",
        _ => "Other",
    };

    println!("{}", result); // Other
}
```
### **Ownership Transfer in Blocks**:
Ownership rules apply to variables declared inside blocks.
```rust
fn main() {
    let x = {
        let y = String::from("Rust");
        y // `y` moves out of this block
    };

    // println!("{}", y); // Error: `y` is not accessible here
    println!("{}", x); // Rust
}
```
### Closure in a block
```rust
// Closure in a block
let result = {
    let multiplier = 2;
    |x| x * multiplier  // Closure capturing multiplier
}(5);  // Immediately invoked
```
### Points to remember:
1. **Expression-Oriented Nature**: 
	- Blocks are powerful because they return values. Use them for concise, inline calculations.
2. **Shadowing**:
	- Variables declared in inner blocks shadow outer variables, which can lead to subtle bugs.
```rust
fn main() {
	let x = 10;
	{
		let x = 20; // Shadows outer `x`
		println!("{}", x); // 20
	}
	println!("{}", x); // 10
}
```
