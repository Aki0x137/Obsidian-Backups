# HashMap
A **`HashMap`** is a collection in Rust that stores key-value pairs. It is part of the **standard library's collections module** and provides efficient lookups, inserts, and deletions based on keys. HashMaps are implemented using a hash table and are particularly useful when you need to associate data with unique keys.
## **1. Creating a HashMap**
The `HashMap` is included in the `std::collections` module, so you need to bring it into scope with `use std::collections::HashMap`.
```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert("Alice", 25);
    map.insert("Bob", 30);
    println!("{:?}", map);
}
```
## **2. Basic Operations**
### **Insert Values**
- Use the `insert` method to add key-value pairs.
- If the key already exists, the value is updated.
```rust
fn main() {
    let mut scores = HashMap::new();
    scores.insert("Team A", 10);
    scores.insert("Team B", 20);
    scores.insert("Team A", 30); // Updates the value for "Team A"
    println!("{:?}", scores);
}
```
### **Access Values**
- Use the `get` method to retrieve a value by its key.
- `get` returns an `Option<&V>` (a reference to the value).
```rust
fn main() {
    let mut map = HashMap::new();
    map.insert("Alice", 25);
    if let Some(age) = map.get("Alice") {
        println!("Alice's age is {}", age);
    } else {
        println!("Key not found");
    }
}
```
### **Remove Values**
- Use the `remove` method to delete a key-value pair.
```rust
fn main() {
    let mut map = HashMap::new();
    map.insert("Alice", 25);
    map.remove("Alice");
    println!("{:?}", map); // Outputs: {}
}
```
## **3. Iterating Over a HashMap**
You can iterate over key-value pairs using a `for` loop.
```rust
fn main() {
    let mut scores = HashMap::new();
    scores.insert("Team A", 10);
    scores.insert("Team B", 20);

    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }
}
```
## **4. Ownership and HashMap**
HashMaps take ownership of their keys and values. This means that any variable moved into a `HashMap` becomes inaccessible outside the `HashMap`.
```rust
fn main() {
    let name = String::from("Alice");
    let mut map = HashMap::new();
    map.insert(name, 25);

    // println!("{}", name); // Error: `name` was moved
}
```
If you need to reuse the key, you can use references:
```rust
fn main() {
    let name = String::from("Alice");
    let mut map = HashMap::new();
    map.insert(&name, 25); // Insert a reference
    println!("{}", name); // `name` is still accessible
}
```
## **5. Default Values**
When accessing a key that may or may not exist, you can use the `entry` API to either retrieve the value or insert a default.
```rust
fn main() {
    let mut map = HashMap::new();
    map.insert("Alice", 25);

    map.entry("Bob").or_insert(30); // Inserts "Bob" with default value 30
    map.entry("Alice").or_insert(40); // Doesn't change "Alice"

    println!("{:?}", map); // Outputs: {"Alice": 25, "Bob": 30}
}
```
## **6. HashMap and Borrowing**
When working with keys in a `HashMap`, Rust allows you to use borrowed types (e.g., `&str`) to access keys even if the HashMap stores owned types (e.g., `String`).
```rust
fn main() {
    let mut map = HashMap::new();
    map.insert(String::from("Alice"), 25);

    if let Some(age) = map.get("Alice") { // Using &str instead of String
        println!("Alice's age is {}", age);
    }
}
```
This is possible because `HashMap` implements the `Borrow` trait for keys.
## **7. HashMap Capacity and Performance**
#### **Capacity**
- HashMaps dynamically grow as more elements are added.
- You can pre-allocate space using `HashMap::with_capacity()` to avoid repeated reallocation.
```rust
fn main() {
    let mut map = HashMap::with_capacity(10); // Preallocate capacity for 10 entries
    map.insert("Alice", 25);
    println!("{:?}", map);
}
```
### **Performance**
- HashMap operations (insert, lookup, delete) are generally **O(1)** on average.
- However, the performance depends on the quality of the hash function and the distribution of keys.
## **8. Custom Hashing**
By default, Rust uses the `std::collections::hash_map::RandomState` hasher, which is secure and resistant to DoS attacks. For custom hashing behavior, you can use the `HashMap::with_hasher()` method.
```rust
use std::collections::HashMap;
use std::hash::{BuildHasherDefault, Hasher};

struct SimpleHasher(u64);

impl Hasher for SimpleHasher {
    fn write(&mut self, bytes: &[u8]) {
        for &b in bytes {
            self.0 ^= b as u64;
        }
    }
    fn finish(&self) -> u64 {
        self.0
    }
}

type SimpleBuildHasher = BuildHasherDefault<SimpleHasher>;

fn main() {
    let mut map: HashMap<_, _, SimpleBuildHasher> = HashMap::default();
    map.insert("Alice", 25);
    println!("{:?}", map);
}
```
### **9. Limitations of HashMap**
1. **Deterministic Order**: HashMaps do not maintain the order of elements. Use `BTreeMap` if you need sorted keys.
2. **Hash Function Overhead**: Hashing introduces overhead compared to other data structures like vectors.
3. **Not Thread-Safe**: Use `std::sync::Mutex` or `dashmap` for concurrent access.
### **10. Summary**

|Feature|Description|
|---|---|
|**Key-Value Pair Storage**|Stores unique keys and associated values.|
|**Fast Lookup**|Average O(1) time complexity for insertions, deletions, and lookups.|
|**Ownership**|Takes ownership of keys and values unless references are used.|
|**Dynamic Resizing**|Automatically grows as elements are added.|
|**Flexible Data Types**|Supports any data type for keys and values (generic over `K` and `V`).|
|**Custom Hashing**|Allows for custom hash functions for specialized use cases.|
|**Default Handling**|Use `entry` API to handle keys that may or may not exist.|
