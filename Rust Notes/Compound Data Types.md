# Strings
There are around 13 different types which can represent strings in Rust. In most of the cases we'll use only 2 types. String `String`  and string slices `&str`.
## String Slice (`&str`)
- Fixed-size string slice (view into string data)
- Immutable by default
- Stored on stack (just the pointer and length)
- Zero-cost abstraction
- Often used for string literals and views into Strings
```rust
// String literals are &'static str 
let literal: &str = "Hello";
// Slicing
let string = String::from("Hello, World!");
let slice: &str = &string[0..5]; // "Hello" 

// Function parameters typically use &str 
fn print_str(s: &str) {
	println!("{}", s);
}
```
## String

- Owned, grow-able string
- Mutable
- Stored on heap
- Has capacity and can grow
- Used when you need to own or modify string data
```rust
// Creating Strings
let mut s = String::new();
let s = String::from("Hello");
let s = "Hello".to_string();

// Modifying 
let mut s = String::from("Hello");
s.push_str(", World!"); // Append string slice
s.push('!'); // Append single char
s.clear(); // Empty the string
```

## Key Differences

1. Memory Management:
```rust
// &str - fixed view
let str_slice: &str = "Hello"; // Lives in binary, static lifetime

// String - owned heap data
let string = String::from("Hello"); // Allocated on heap
```

2. Mutability:
```rust
let mut s = String::from("Hello"); s.push_str(" World"); // Works let s: &str = "Hello"; // s.push_str(" World"); // Won't compile - &str is immutable
```
3. Memory Layout:
```rust
// &str is a fat pointer (16 bytes on 64-bit systems)
// Contains:
// - pointer to data
// - length
let slice: &str = "Hello";

// String contains (24 bytes on 64-bit systems):
// - pointer to heap data
// - length
// - capacity
let string = String::from("Hello");
```
4. Common Conversions:
```rust
// &str to String
let slice: &str = "Hello";
let string: String = slice.to_string();
let string: String = String::from(slice);

// String to &str
let string: String = String::from("Hello");
let slice: &str = &string;
let slice: &str = string.as_str();
```
5. Usage Patterns:
```rust
// Function accepting both String and &str
fn print_str(s: &str) {
    println!("{}", s);
}

let string = String::from("Hello");
let slice = "World";

print_str(&string); // Works - String derefs to &str
print_str(slice);   // Works directly

// Builder pattern with String
let mut builder = String::with_capacity(100);
builder.push_str("Hello");
builder.push_str(", ");
builder.push_str("World!");
```
6. Performance Considerations:
```rust
// &str - No allocation, just borrowing
fn process_str(s: &str) -> &str {
    &s[1..4]
}

// String - Owns data, can be more expensive
fn process_string(s: String) -> String {
    s + "suffix"
}
```
7. Common Patterns:
```rust
// Accept &str as parameter (more flexible)
fn process(s: &str) -> String {
    s.to_uppercase()
}

// Return String when ownership is needed
fn generate() -> String {
    String::from("Generated content")
}

// Use &str for string literals
const GREETING: &str = "Hello, World!";
```
8. Working with collections:
```rust
// Vec of strings
let v: Vec<String> = vec![String::from("Hello")];

// Vec of string slices (needs explicit lifetime)
let v: Vec<&str> = vec!["Hello"];

// HashMap with string keys
use std::collections::HashMap;
let mut map: HashMap<String, i32> = HashMap::new();
map.insert(String::from("key"), 42);
```

**The general rule of thumb is:**
- Use `&str` for function parameters (more flexible)
- Use `String` when you need to own or modify the string
- Use `&str` for string literals and slices
- Convert to `String` when you need ownership or modification capabilities