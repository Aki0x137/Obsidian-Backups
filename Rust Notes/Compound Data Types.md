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

# Arrays

- Fixed length, known at compile time
- Stored entirely on stack
- Zero runtime overhead
- Cannot grow or shrink
```rust
// Declaration and initialization
let arr: [i32; 5] = [1, 2, 3, 4, 5];
let zeros = [0; 10];  // Initialize 10 elements with 0

// Accessing elements
let first = arr[0];  // Direct index access
let slice = &arr[1..4];  // Slicing

// Size must be known at compile time
const SIZE: usize = 10;
let arr: [i32; SIZE] = [0; SIZE];  // Works with const
```

## Use cases for Arrays:

1. Fixed-size buffers:
```rust
let buffer: [u8; 1024] = [0; 1024];  // 1KB buffer
```
2. Small, known collections:
```rust
let rgba: [u8; 4] = [255, 128, 0, 255];  // Color values
let years: [i32; 12] = [2010, 2011, 2012, 2013, 2014, 2015, 
                       2016, 2017, 2018, 2019, 2020, 2021];
```
3. Performance critical code:
```rust
// No heap allocation, very fast
fn process_fixed_data(data: [f64; 16]) {
    for item in data {
        // Process fixed-size chunks
    }
}
```
# Vectors

- Dynamic length, grows at runtime
- Stored on heap (with stack pointer)
- Has capacity management
- Can grow and shrink
```rust
// Creation
let mut vec: Vec<i32> = Vec::new();
let vec = vec![1, 2, 3, 4, 5];
let vec = Vec::with_capacity(10);

// Modification
vec.push(6);
vec.pop();
vec.insert(0, 42);
vec.remove(2);
vec.extend([7, 8, 9].iter());

// Accessing elements
let first = vec[0];  // Can panic
let first = vec.get(0);  // Returns Option<&T>
let slice = &vec[1..4];
```

## Use cases for Vectors:

1. Dynamic collections:
```rust
let mut numbers = Vec::new();
while let Some(num) = get_next_number() {
    numbers.push(num);
}
```
2. Building lists at runtime:
```rust
fn read_lines(file: &str) -> Vec<String> {
    let mut lines = Vec::new();
    // Read file and push lines
    lines
}
```
3. Complex data structures:
```rust
struct Graph {
    edges: Vec<Vec<usize>>,  // Adjacency list
}
```
## Comparison and Specific Use Cases
1. Memory Layout:
```rust
   // Array - Contiguous stack memory
let arr: [i32; 4] = [1, 2, 3, 4];
// |1|2|3|4| on stack

// Vector - Stack pointer to heap data
let vec = vec![1, 2, 3, 4];
// Stack: |ptr|len|cap| -> Heap: |1|2|3|4|
```
2. Performance Considerations:
```rust
// Arrays - Best for small, fixed sizes
fn process_coordinates(point: [f64; 3]) {
    // Fast, no indirection
}

// Vectors - Best for variable sizes
fn process_points(points: &Vec<[f64; 3]>) {
    // More flexible but has heap allocation
}
```
3. Common Patterns:
```rust
   // Array for fixed-size lookup tables
const DAYS: [&str; 7] = ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"];

// Vector for dynamic content
let mut log_entries = Vec::new();
for event in events {
    log_entries.push(event);
}
```
4. Memory Management:
```rust
// Preallocated array on stack
let buffer: [u8; 1024] = [0; 1024];

// Growing vector with capacity
let mut vec = Vec::with_capacity(1024);
// Grows automatically when needed
```
5. Iteration Performance:
```rust
// Array - Very fast iteration
for x in [1, 2, 3, 4, 5] {
    // Direct iteration, no pointer dereferencing
}

// Vector - Slightly more overhead
for x in &vec {
    // Requires following pointer to heap
}
```
## General Guidelines:
- Use arrays when:
    - Size is known at compile time
    - Small, fixed-size collections
    - Maximum performance is needed
    - Working with embedded systems or no-std
    - Stack allocation is preferred
- Use vectors when:
    - Size needs to change
    - Size is not known until runtime
    - Need to own a variable-sized collection
    - Building data structures
    - Working with dynamic content
# Tuples
A tuple is a fixed-size ordered collection that can hold multiple values of different types.
```rust
// Basic tuple
let tuple: (i32, String, bool) = (42, String::from("hello"), true);

// Type annotation is optional due to inference
let tuple = (42, String::from("hello"), true);

// Accessing elements using dot notation
println!("{}", tuple.0);  // 42
println!("{}", tuple.1);  // "hello"
```

## Pattern Matching and Destructuring
```rust
// Destructuring
let (number, text, flag) = tuple;

// Partial destructuring with _
let (number, _, flag) = tuple;

// In match expressions
match tuple {
    (42, _, true) => println!("Found specific pattern"),
    (x, _, _) if x > 50 => println!("Large number"),
    _ => println!("No match"),
}
```

## Unit Type (Empty Tuple)
