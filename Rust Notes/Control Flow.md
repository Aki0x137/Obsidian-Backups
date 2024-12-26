# The `loop` Loop
The `loop` construct creates an **infinite loop**. It is often used for tasks that run indefinitely, like servers, or when the termination condition is checked explicitly within the loop.
Syntax:
```rust
loop {
    // Code to execute
}
```
Example:
```rust
fn main() {
    let mut counter = 0;

    loop {
        counter += 1;
        println!("Counter: {}", counter);

        if counter == 5 {
            break; // Exit the loop
        }
    }
    println!("Loop terminated");
}
```
### **Key Features**
1. **Infinite by Default**: Runs forever unless explicitly terminated with `break`.
2. **Flexible Control**: Allows custom exit conditions inside the body.
3. **Returning Values from `loop`**: A `loop` can return a value when combined with `break`.
```rust
fn main() {
    let result = loop {
        let x = 5;
        break x * 2; // Break with a value
    };
    println!("Result: {}", result);
}
```

# The `while` Loop
The `while` loop runs as long as a specified condition evaluates to `true`. It is useful when the number of iterations depends on dynamic conditions.
Syntax:
```rust
while condition {
    // Code to execute
}
```
Example:
```rust
fn main() {
    let mut n = 5;

    while n > 0 {
        println!("Countdown: {}", n);
        n -= 1;
    }

    println!("Liftoff!");
}
```
### **Key Features**
1. **Condition-Based Iteration**: Executes only when the condition is true.
2. **Dynamic Control**: The loop condition is evaluated before each iteration.
3. **Risk of Infinite Loops**: Ensure the condition eventually becomes `false` to avoid infinite loops.
# The `for` Loop
The `for` loop iterates over a **range** or an **iterator**. It is the most commonly used loop in Rust due to its simplicity and safety.
Syntax:
```rust
for variable in iterable {
    // Code to execute
}
```
Example:
- Iterating over a range:
```rust
fn main() {
    for i in 1..5 {
        println!("Value: {}", i); // 1, 2, 3, 4
    }

    for i in 1..=5 {
        println!("Value: {}", i); // 1, 2, 3, 4, 5
    }
}
```
- Iterating over an array:
```rust
fn main() {
    let array = [10, 20, 30, 40];

    for value in array {
        println!("Value: {}", value);
    }
}
```
- Iterating with `enumerate`:
```rust
fn main() {
    let fruits = ["apple", "banana", "cherry"];

    for (index, fruit) in fruits.iter().enumerate() {
        println!("Index: {}, Fruit: {}", index, fruit);
    }
}
```