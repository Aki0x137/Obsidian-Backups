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
