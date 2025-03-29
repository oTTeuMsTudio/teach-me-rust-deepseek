In Rust, **ownership** and **borrowing** are key concepts that help manage memory safely without needing a garbage collector. Here's how borrowing works in terms of permissions:

---

### 1. **Ownership Basics**
- When you own something (like a variable), you have full control over it. You can:
  - **Read** its value.
  - **Write** to change its value.
  - **Own** it, meaning you decide what happens to it (e.g., pass it around or destroy it).

For example:
```rust
let mut x = 5; // You "own" `x`
x = 10;        // You can write to it because you own it
println!("{}", x); // You can read it because you own it
```

---

### 2. **Borrowing (Creating a Reference)**
When you borrow something in Rust, you're temporarily lending access to it without transferring ownership. Borrowing is done using references (`&` for immutable references, `&mut` for mutable references).

#### What Happens When You Borrow?
- When you create a reference (borrow), Rust enforces rules to ensure safety:
  - If you borrow immutably (`&`), you can **read** the value but **cannot write** to it.
  - If you borrow mutably (`&mut`), you can both **read** and **write** the value.

However, when you borrow, Rust temporarily restricts your ability to use the original variable to prevent conflicts. This is where the idea of "losing permissions" comes in.

---

### 3. **Permissions: Read, Write, Own**
Rust tracks three kinds of permissions for a variable:
- **Read**: You can look at the value.
- **Write**: You can change the value.
- **Own**: You control the variable's lifetime and can move or destroy it.

When you borrow, Rust adjusts these permissions based on the type of borrow:

#### Case 1: Immutable Borrow (`&`)
- When you create an immutable reference (`&x`), you gain:
  - **Read** permission for the reference.
- But you lose:
  - **Write** permission for the original variable (`x`).
  - Why? Because allowing writes while someone else is reading could lead to inconsistent data.

Example:
```rust
let mut x = 5;
let y = &x; // Borrow immutably
// x = 10; // Error: Cannot write to `x` while `y` is borrowing it
println!("{}", *y); // OK: Can read through `y`
```

#### Case 2: Mutable Borrow (`&mut`)
- When you create a mutable reference (`&mut x`), you gain:
  - **Read** and **Write** permission for the reference.
- But you lose:
  - **Write** permission for the original variable (`x`).
  - **Read** permission for the original variable (`x`) if another reference exists.
  - Why? Because allowing multiple accesses to the same data could cause race conditions or undefined behavior.

Example:
```rust
let mut x = 5;
let y = &mut x; // Borrow mutably
*y = 10;        // OK: Can write through `y`
// println!("{}", x); // Error: Cannot read `x` while `y` is borrowing it
```

---

### 4. **Why Does This Happen?**
Rust enforces these rules to ensure memory safety:
- Preventing simultaneous reads and writes avoids data races.
- Restricting ownership ensures there’s no confusion about who controls the data.

By temporarily restricting permissions, Rust guarantees that your program behaves predictably and avoids bugs like:
- Reading invalid data.
- Writing to the same data from multiple places at once.

---

### Summary in Simple Terms
- When you borrow (create a reference):
  - You gain **read** and/or **write** permission for the reference.
  - You lose certain permissions for the original variable:
    - For immutable borrows (`&`), you lose **write** permission.
    - For mutable borrows (`&mut`), you lose both **read** and **write** permissions for the original variable while the borrow is active.

This system ensures safe and predictable memory management without runtime overhead.

---

### Final Answer
**When you borrow in Rust, you gain read/write permissions for the reference but lose write permissions for the original variable. For mutable borrows, you also lose read permissions for the original variable while the borrow is active.**
