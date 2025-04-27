# In Rust, strings are UTF-8 encoded by default. 
This means that Rust's `String` and `&str` types are designed to handle Unicode text seamlessly. Understanding how to store and manipulate UTF-8 encoded text is fundamental to working with strings in Rust. Lets go through the key concepts and techniques for working with UTF-8 encoded text in Rust.

---

### 1. **Understanding UTF-8 Encoding**
UTF-8 is a variable-width character encoding that can represent any Unicode character. It is backward-compatible with ASCII, meaning that ASCII characters (0–127) are represented using a single byte, while other characters may use multiple bytes.

Rust's `String` type is a growable, heap-allocated data structure that stores UTF-8 encoded text. The `&str` type is a string slice, which is a reference to a portion of a `String`.

---

### 2. **Creating a UTF-8 Encoded String**
You can create a `String` in Rust using various methods:

#### a. Using the `String::from` function:
```rust
let s = String::from("Hello, 世界"); // "世界" means "world" in Chinese
println!("{}", s);
```

#### b. Using the `to_string` method:
```rust
let s = "Hello, 世界".to_string();
println!("{}", s);
```

#### c. Using a string literal (`&str`):
```rust
let s: &str = "Hello, 世界";
println!("{}", s);
```

All of these methods store the text as UTF-8 encoded.

---

### 3. **Manipulating Strings**
Rust provides several methods to manipulate strings while ensuring they remain valid UTF-8.

#### a. Appending to a `String`:
You can append text to a `String` using the `push_str` or `push` methods:
```rust
let mut s = String::from("Hello, ");
s.push_str("世界!"); // Append a string slice
s.push('!');        // Append a single character
println!("{}", s);
```

#### b. Concatenating Strings:
You can concatenate strings using the `+` operator or the `format!` macro:
```rust
let s1 = String::from("Hello, ");
let s2 = String::from("世界");
let s3 = s1 + &s2; // `s1` is moved, and `s2` is borrowed
println!("{}", s3);

let s4 = format!("{}{}", s3, "!"); // No ownership is taken
println!("{}", s4);
```

---

### 4. **Iterating Over Characters**
Since UTF-8 is a variable-width encoding, you cannot index into a `String` directly (e.g., `s[0]` is invalid). Instead, you can iterate over the characters using the `chars` method:
```rust
let s = String::from("你好，世界");
for c in s.chars() {
    println!("{}", c);
}
```

If you need to work with bytes instead of characters, you can use the `bytes` method:
```rust
let s = String::from("你好，世界");
for b in s.bytes() {
    println!("{}", b);
}
```

---

### 5. **Handling Invalid UTF-8**
Rust ensures that all `String` instances contain valid UTF-8. If you encounter invalid UTF-8 data (e.g., from external sources), you can use the `std::str::from_utf8` function to validate it:
```rust
let bytes = vec![0xe4, 0xbd, 0xa0, 0xe5, 0xa5, 0xbd]; // UTF-8 bytes for "你好"
match std::str::from_utf8(&bytes) {
    Ok(s) => println!("Valid UTF-8: {}", s),
    Err(e) => println!("Invalid UTF-8: {}", e),
}
```

If you need to work with potentially invalid UTF-8 data, you can use the `Vec<u8>` type or the `std::ffi::OsStr` type.

---

### 6. **String Slicing**
When slicing a `String`, you must ensure that the slice boundaries align with valid UTF-8 character boundaries. Otherwise, Rust will panic at runtime:
```rust
let s = String::from("你好，世界");
let hello = &s[0..9]; // Valid: "你好，"
println!("{}", hello);

// let invalid = &s[0..8]; // Invalid: Panics because it cuts a multi-byte character
```

To avoid panics, you can use the `char_indices` method to safely find valid slicing points:
```rust
let s = String::from("你好，世界");
let mut indices = s.char_indices();
if let Some((_, _)) = indices.nth(2) {
    let slice = &s[..indices.nth(2).unwrap().0];
    println!("{}", slice);
}
```

---

### 7. **Common Pitfalls**
- **Indexing:** Rust does not allow direct indexing into a `String` because UTF-8 characters may span multiple bytes.
- **Ownership:** Be mindful of ownership when working with `String` and `&str`. For example, using the `+` operator moves the left-hand operand.
- **Invalid UTF-8:** Always validate UTF-8 data from external sources before converting it to a `String`.

---

### 8. **Example: Putting It All Together**
Here’s a complete example that demonstrates creating, manipulating, and iterating over a UTF-8 encoded string:
```rust
fn main() {
    // Create a UTF-8 encoded string
    let mut greeting = String::from("Hello, ");
    greeting.push_str("世界"); // Append Chinese characters
    greeting.push('!');      // Append an exclamation mark

    // Print the string
    println!("{}", greeting);

    // Iterate over characters
    println!("Characters:");
    for c in greeting.chars() {
        println!("{}", c);
    }

    // Iterate over bytes
    println!("Bytes:");
    for b in greeting.bytes() {
        println!("{}", b);
    }
}
```

---

### Summary
- Rust's `String` and `&str` types are UTF-8 encoded by default.
- Use `String::from`, `to_string`, or string literals to create strings.
- Manipulate strings with methods like `push_str`, `push`, and `format!`.
- Iterate over characters with `chars` or bytes with `bytes`.
- Handle invalid UTF-8 carefully using `std::str::from_utf8`.
- Avoid direct indexing into strings due to UTF-8's variable-width nature.

By mastering these concepts, you can confidently work with UTF-8 encoded text in Rust.
