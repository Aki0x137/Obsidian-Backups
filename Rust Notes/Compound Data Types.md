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
### **Key Features of Tuples**
1. **Fixed Size**: The size of a tuple is determined at compile time and cannot change.
2. **Heterogeneous Types**: Each element in a tuple can be of a different type.
3. **Indexed Access**: Tuple elements are accessed using zero-based indexing.
4. **Destructuring**: You can unpack a tuple into its individual components.

A tuple is defined using parentheses `()` with comma-separated values.
```rust
// Basic tuple
let tuple: (i32, String, bool) = (42, String::from("hello"), true);

// Type annotation is optional due to inference
let tuple = (42, String::from("hello"), true);

// Accessing elements using dot notation
println!("{}", tuple.0);  // 42
println!("{}", tuple.1);  // "hello"
```
## **Accessing Tuple Elements**
You can access individual elements of a tuple using the dot (`.`) operator followed by the index.
```rust
fn main() {
    let my_tuple = (10, 3.14, "Rust");

    println!("First element: {}", my_tuple.0); // 10
    println!("Second element: {}", my_tuple.1); // 3.14
    println!("Third element: {}", my_tuple.2); // Rust
}
```
## Destructuring Tuples
```rust
fn main() {
    let my_tuple = (10, 3.14, "Rust");

    let (x, y, z) = my_tuple;

    println!("x: {}, y: {}, z: {}", x, y, z); // x: 10, y: 3.14, z: Rust
}
```
## Unit Type (Empty Tuple)
```rust
// Unit type is an empty tuple
let unit: () = ();

// Common in functions that don't return values
fn do_something() -> () {
    // Function body
}

// Equivalent to
fn do_something() {
    // Function body
}
```
## Tuple Structs
```rust
// Named tuples without named fields
struct Point(i32, i32);
struct Color(u8, u8, u8);

let point = Point(10, 20);
let color = Color(255, 0, 0);
```
## Common Use Cases
1. Returning multiple values:
```rust
fn divide(dividend: i32, divisor: i32) -> (i32, i32) {
    let quotient = dividend / divisor;
    let remainder = dividend % divisor;
    (quotient, remainder)
}

let (quot, rem) = divide(10, 3);
```
2. Temporary grouping:
```rust
fn process_coordinates() {
    let coordinates = vec![(0, 1), (2, 3), (4, 5)];
    
    for (x, y) in coordinates {
        println!("Point at ({}, {})", x, y);
    }
}
```
3. Simple key-value pairs:
```rust
let pairs = vec![
    ("name", "Alice"),
    ("age", "30"),
    ("city", "London")
];
```
## Working with References
```rust
// Tuple of references
let tuple: (&str, &i32) = ("hello", &42);

// Reference to tuple
let tuple_ref: &(String, i32) = &(String::from("hello"), 42);
```


**Best Practices:**
- Use tuples for simple groupings of 2-3 values
- Consider structs for larger groupings or when field names add clarity
- Use tuple structs when you want type safety but don't need field names
- Prefer destructuring over index access for clarity
- Remember tuples are fixed-size and ordered

## **Basic Tuple Pattern Matching**
Pattern matching in tuples is a powerful and elegant feature in Rust, allowing you to decompose and extract values from tuples easily. Rust's pattern matching works by comparing a tuple's structure against a specified pattern. If the pattern matches, the elements of the tuple are extracted, and you can use them directly in your code.
Pattern matching for tuples uses `match` statements or `if let` syntax to destructure the tuple.
```rust
fn main() {
    let point = (3, 5);

    match point {
        (0, 0) => println!("The point is at the origin."),
        (x, 0) => println!("The point lies on the X-axis at x = {}.", x),
        (0, y) => println!("The point lies on the Y-axis at y = {}.", y),
        (x, y) => println!("The point is at ({}, {}).", x, y),
    }
}
```
**Explanation**:
- `(0, 0)` matches the origin.
- `(x, 0)` matches points on the X-axis; the `x` value is captured.
- `(0, y)` matches points on the Y-axis; the `y` value is captured.
- `(x, y)` is the fallback pattern, capturing all other cases.
### **Destructuring Nested Tuples**
Pattern matching supports nested tuples, allowing you to destructure inner values.
```rust
fn main() {
    let nested_tuple = ((1, 2), (3, 4));

    match nested_tuple {
        ((x1, y1), (x2, y2)) => println!("Points are ({}, {}) and ({}, {}).", x1, y1, x2, y2),
    }
}
```
### **Using Ranges in Patterns**
You can use ranges to match values within a range.
```rust
fn main() {
    let point = (5, 7);

    match point {
        (0..=5, 0..=5) => println!("Point is within the lower-left quadrant."),
        (6..=10, 6..=10) => println!("Point is within the upper-right quadrant."),
        _ => println!("Point is somewhere else."),
    }
}
```
**Explanation**:
- `0..=5` matches numbers between 0 and 5 inclusive.
- Patterns can include ranges for each tuple element.
### **Using `if` Guards with Patterns**
`if` guards add extra conditions to patterns for finer-grained matching.
```rust
fn main() {
    let point = (5, -10);

    match point {
        (x, y) if x == y => println!("Point lies on the diagonal."),
        (x, y) if x > 0 && y > 0 => println!("Point is in the first quadrant."),
        _ => println!("Point is elsewhere."),
    }
}
```
**Explanation**:
- `if x == y` checks if the point lies on the diagonal.
- Guards allow conditional logic on top of structural matching
### **Tuple Matching in `if let`**
The `if let` syntax is useful for matching one specific pattern.
```rust
fn main() {
    let coords = (0, 5);

    if let (0, y) = coords {
        println!("Point lies on the Y-axis at y = {}.", y);
    }
}
```
**Explanation**:
- `if let` destructures the tuple only if it matches `(0, y)`.
### **Error Handling with Tuples**:
Combine an error code and message in a tuple and match based on code.
```rust
let error = (404, "Not Found");

match error {
    (200, _) => println!("Success"),
    (404, _) => println!("Page not found"),
    (_, message) => println!("Error: {}", message),
}
```
### **Iterating with Tuple-Based Iterators**:
Destructure `(index, value)` pairs from an iterator.
```rust
let v = vec!["a", "b", "c"];
for (i, val) in v.iter().enumerate() {
    println!("Index: {}, Value: {}", i, val);
}
```
#### **Key Points to Remember**
- Patterns must match the **structure and size** of the tuple.
- Use `_` to ignore elements you don’t need.
- Nested tuples and guards allow complex and precise matching.
- Use `if let` when you care about a single pattern.