# In Rust, the module system is a powerful way to organize code. 
To access items (functions, structs, enums, traits, etc.) defined in different parts of the module tree, you need to use **paths**. Paths are used to refer to these items either by their absolute location or relative to the current module.

Here’s a detailed explanation of how paths work in Rust:

---

### **1. Module Tree Basics**
Before diving into paths, let's briefly review the module tree:
- The root of the module tree is the `crate` keyword.
- Modules can be nested within other modules.
- Items (functions, structs, etc.) are defined inside modules.

For example:
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}
```

Here, the module tree looks like this:
```
crate
 └── front_of_house
      └── hosting
           └── add_to_waitlist
```

To access `add_to_waitlist`, you need to specify its path.

---

### **2. Types of Paths**
There are two types of paths in Rust:
1. **Absolute Path**: Starts from the crate root using the `crate` keyword.
2. **Relative Path**: Starts from the current module and uses `self`, `super`, or an identifier in the current module.

#### **Example of Absolute Path**
```rust
crate::front_of_house::hosting::add_to_waitlist();
```
This path starts from the root of the crate (`crate`) and navigates down to the `add_to_waitlist` function.

#### **Example of Relative Path**
If you are already inside the `front_of_house` module, you can use a relative path:
```rust
self::hosting::add_to_waitlist();
```
Or, if you are inside the `hosting` module:
```rust
add_to_waitlist();
```

---

### **3. Using `use` to Bring Paths into Scope**
To avoid typing long paths repeatedly, you can use the `use` keyword to bring items into scope. This makes the code cleaner and more readable.

#### **Example with `use`**
```rust
use crate::front_of_house::hosting;

fn main() {
    hosting::add_to_waitlist();
}
```
Here, the `use` statement brings the `hosting` module into scope, so you can directly call `hosting::add_to_waitlist()`.

#### **Idiomatic Use of `use`**
When bringing functions into scope, it’s idiomatic to bring the parent module into scope rather than the function itself. This makes it clear where the function is coming from.

However, for structs, enums, and other types, it’s idiomatic to bring the type itself into scope:
```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

---

### **4. Super and Self Keywords**
- `self`: Refers to the current module.
- `super`: Refers to the parent module.

#### **Example of `super`**
```rust
mod back_of_house {
    fn fix_incorrect_order() {
        super::serve_order();
    }
}

fn serve_order() {}
```
Here, `super` is used to refer to the `serve_order` function in the parent module.

#### **Example of `self`**
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {
            self::seat_at_table();
        }

        fn seat_at_table() {}
    }
}
```
Here, `self` refers to the `hosting` module.

---

### **5. Re-exporting with `pub use`**
By default, items brought into scope with `use` are private. If you want to make them public to other modules, you can use `pub use`.

#### **Example of `pub use`**
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

fn main() {
    hosting::add_to_waitlist();
}
```
Here, `pub use` re-exports the `hosting` module, making it accessible to external code.

---

### **6. Common Pitfalls**
- **Private Items**: By default, items in a module are private. You need to explicitly mark them as `pub` to make them accessible outside the module.
- **Path Conflicts**: If two items have the same name, you may encounter conflicts. Use aliases to resolve this:
  ```rust
  use std::fmt::Result as FmtResult;
  use std::io::Result as IoResult;
  ```

---

### **Summary**
- **Absolute Path**: Starts with `crate` and navigates through the module tree.
- **Relative Path**: Starts with `self`, `super`, or an identifier in the current module.
- **Use `use`**: Bring items into scope for cleaner code.
- **Re-export with `pub use`**: Make items accessible to external modules.
- **Handle Privacy**: Use `pub` to expose items to other modules.

---

### **Practice Example**
Here’s a complete example to solidify your understanding:
```rust
mod restaurant {
    pub mod front_of_house {
        pub mod hosting {
            pub fn add_to_waitlist() {
                println!("Customer added to waitlist!");
            }
        }
    }

    pub fn eat_at_restaurant() {
        // Absolute path
        crate::restaurant::front_of_house::hosting::add_to_waitlist();

        // Relative path
        self::front_of_house::hosting::add_to_waitlist();
    }
}

fn main() {
    restaurant::eat_at_restaurant();
}
```
