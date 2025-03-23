# Ownership recap
Understanding **ownership** is crucial in Rust because it's one of the language's core features that ensures memory safety without a garbage collector. Ownership helps prevent common programming errors like null pointer dereferencing, data races, and memory leaks.

Here’s a recap of ownership in Rust:

---

### **Key Concepts of Ownership**

1. **Ownership Rules**
   - Each value in Rust has an **owner**.
   - There can only be **one owner** at a time.
   - When the owner goes out of scope, the value is dropped (deallocated).

2. **Scope**
   - A scope defines the region of code where a variable is valid.
   - Once a variable goes out of scope, Rust automatically calls the `drop` function to clean up its resources.

   ```rust
   {
       let s = String::from("hello"); // s comes into scope
       println!("{}", s);            // s is valid here
   } // s goes out of scope, memory is freed
   ```

3. **Move Semantics**
   - In Rust, when you assign a value to another variable or pass it to a function, the ownership is **moved** (not copied).
   - After the move, the original variable becomes invalid.

   ```rust
   let s1 = String::from("hello");
   let s2 = s1; // Ownership of the string is moved from s1 to s2
   // println!("{}", s1); // Error: s1 is no longer valid
   println!("{}", s2); // s2 owns the string now
   ```

4. **Clone**
   - If you want to deeply copy the data (heap + stack), you can use the `clone` method.
   - This creates a new instance of the data.

   ```rust
   let s1 = String::from("hello");
   let s2 = s1.clone(); // Deep copy of the string
   println!("s1 = {}, s2 = {}", s1, s2); // Both are valid
   ```

5. **Copy Trait**
   - Some types implement the `Copy` trait, meaning they are copied instead of moved when assigned or passed around.
   - These are typically simple types like integers, booleans, and floats.

   ```rust
   let x = 5;
   let y = x; // x is copied, not moved
   println!("x = {}, y = {}", x, y); // Both are valid
   ```

6. **Borrowing**
   - Instead of transferring ownership, you can lend (borrow) access to a value using references (`&`).
   - Borrowing does not transfer ownership, so the original owner remains valid.

   ```rust
   fn main() {
       let s1 = String::from("hello");
       let len = calculate_length(&s1); // Borrow s1 with a reference
       println!("The length of '{}' is {}.", s1, len);
   }

   fn calculate_length(s: &String) -> usize { // Takes a reference
       s.len()
   }
   ```

7. **Mutable References**
   - You can borrow a mutable reference (`&mut`) to modify the data.
   - However, there can only be **one mutable reference** to a piece of data at a time to prevent data races.

   ```rust
   let mut s = String::from("hello");
   change(&mut s); // Pass a mutable reference
   println!("{}", s);

   fn change(s: &mut String) {
       s.push_str(", world");
   }
   ```

8. **Dangling References**
   - Rust prevents dangling references by enforcing that references must always be valid.
   - The compiler ensures that data referenced by a borrowed value cannot go out of scope before the reference itself.

   ```rust
   fn main() {
       let reference_to_nothing = dangle(); // Compiler error
   }

   fn dangle() -> &String {
       let s = String::from("hello");
       &s // Error: s goes out of scope, so the reference is invalid
   }
   ```

9. **Slice Types**
   - Slices are references to a portion of a collection, such as a string slice (`&str`) or an array slice (`&[T]`).
   - They do not own the data but provide a view into it.

   ```rust
   let s = String::from("hello world");
   let hello = &s[0..5]; // Reference to part of the string
   let world = &s[6..11];
   println!("{} {}", hello, world);
   ```

---

### **Ownership Recap Summary**

- **Ownership**: Ensures memory safety by managing who owns a value and when it should be deallocated.
- **Move**: Transfers ownership of a value, invalidating the original variable.
- **Clone**: Creates a deep copy of the data.
- **Borrowing**: Allows temporary access to data without transferring ownership.
  - Immutable references (`&T`) allow multiple readers.
  - Mutable references (`&mut T`) allow one writer but no simultaneous readers.
- **Dangling References**: Rust prevents them by ensuring references are always valid.
- **Slices**: Provide a way to reference a contiguous subset of elements in a collection.

---

### **Why Ownership Matters**
- **Memory Safety**: Prevents issues like null pointers, dangling references, and double frees.
- **Concurrency**: Helps avoid data races by enforcing strict borrowing rules.
- **Performance**: Avoids runtime overhead from garbage collection while maintaining safety.

By mastering ownership, you’ll be able to write safe, efficient, and idiomatic Rust code. Practice these concepts with small programs to internalize them!

---

### **Example Exercise**
Try running this code and observe how ownership works:

```rust
fn main() {
    let s1 = String::from("Rust");
    let s2 = s1; // Move occurs here
    // println!("{}", s1); // Error: s1 is invalid
    println!("{}", s2); // s2 owns the string

    let s3 = s2.clone(); // Clone the string
    println!("s2 = {}, s3 = {}", s2, s3); // Both are valid

    let len = calculate_length(&s3); // Borrow s3
    println!("The length of '{}' is {}.", s3, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

This exercise demonstrates moves, clones, and borrowing in action. Experiment with it to solidify your understanding!
