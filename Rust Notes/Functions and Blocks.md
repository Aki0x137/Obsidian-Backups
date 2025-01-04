# Functions

- The last expression in a function is its return value (no `return` keyword needed).
- Function Name must be snake_case by convention.
```rust
// Basic function
fn greet(name: &str) -> String {
    format!("Hello, {}!", name)  // Implicit return without semicolon
}

// Function with multiple parameters
fn add(a: i32, b: i32) -> i32 {
    a + b  // Expression without semicolon is returned
}

// Unit return type
fn log(message: &str) {  // -> () is implicit
    println!("{}", message);
}
```
## Early Return
Use the `return` statement to return a value early
```rust
// Early return
fn process_number(n: i32) -> Option<i32> {
    if n < 0 {
        return None;  // Early return
    }
    Some(n * 2)  // Implicit return
}

// Multiple return points
fn check_value(x: i32) -> &'static str {
    if x < 0 {
        "negative"
    } else if x > 0 {
        "positive"
    } else {
        "zero"
    }
}
```

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

// Mutable borrowing
fn modify_string(s: &mut String) {
	s.push_str(" World");
}

fn main() {
    let s = String::from("Rust");
    takes_ownership(s); // `s` is now invalid

    let t = String::from("Borrowing");
    borrows_value(&t); // `t` is still valid
}
```
### Function Pointers and Type Aliases
```rust
// Function pointer type
type Operation = fn(i32, i32) -> i32;

fn apply_operation(a: i32, b: i32, op: Operation) -> i32 {
    op(a, b)
}

// Using function pointers
fn add(a: i32, b: i32) -> i32 { a + b }
fn multiply(a: i32, b: i32) -> i32 { a * b }

let result1 = apply_operation(5, 3, add);
let result2 = apply_operation(5, 3, multiply);
```

## Diverging Functions
A **diverging function** in Rust is a special kind of function that never returns to the caller. Instead of having a return type like `i32` or `()`, diverging functions are declared with the return type `!` (pronounced [[Data Type Size#**The `!` Type**|never]]).
### Key Characteristics of Diverging Functions
1. **Never Return**:
    - These functions either loop indefinitely, terminate the program, or cause the thread to panic.
2. **Return Type `!`**:
    - The `!` type indicates that the function does not return any value, not even `()`.
3. **Used in Special Scenarios**:
    - Error handling (`panic!`).
    - Infinite loops.
    - Abnormal termination of the program.
### Use Cases for Diverging Functions
1. **Error Handling**: Diverging functions like `panic!` are used for unrecoverable errors.
```rust
fn get_config_value() -> i32 {
    panic!("Configuration file is missing!");
}
```
2. **Unreachable Code**: The `unreachable!` macro is a diverging function used to mark code that should logically never be reached.
```rust
fn match_example(value: i32) -> i32 {
    match value {
        0 => 10,
        1 => 20,
        _ => unreachable!("Invalid value!"),
    }
}
```
3. **Infinite Loops**: Long-running or server-like processes often use infinite loops.
```rust
fn run_server() -> ! {
    loop {
        println!("Server is running...");
    }
}
```
4. **Abnormal Program Termination**: To exit the program from a specific point.
```rust
fn exit_on_error() -> ! {
    println!("Critical error occurred!");
    std::process::exit(1);
}
```
### Behavior in Control Flow
Because diverging functions never return, they can be used in places where a function is expected to "end" a control flow branch.
```rust
fn handle_input(input: Option<&str>) {
    match input {
        Some(value) => println!("Input: {}", value),
        None => panic!("No input provided!"), // Diverging function handles this branch
    }
}
```

---
# Blocks
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
### Ownership Transfer in Blocks:
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
