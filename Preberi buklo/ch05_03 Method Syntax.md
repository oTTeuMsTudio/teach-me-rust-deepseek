# Method syntax
In Rust, **method syntax** is used to define and call functions that are associated with a particular type, such as a `struct`, `enum`, or `trait`. Methods are defined within an `impl` block (short for "implementation") and are called using dot notation (`.`) on an instance of the type. This makes them closely tied to the data they operate on.

Let’s break this down step by step:

---

### 1. **Defining Methods**
Methods are defined inside an `impl` block. The first parameter of a method is always `self`, which refers to the instance of the type the method is being called on. There are three common ways to use `self`:
- `&self`: A reference to the instance (borrowing immutably).
- `&mut self`: A mutable reference to the instance (borrowing mutably).
- `self`: Takes ownership of the instance.

#### Example: Defining a Struct and Its Methods
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Method that borrows `self` immutably
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // Method that borrows `self` mutably
    fn scale(&mut self, factor: u32) {
        self.width *= factor;
        self.height *= factor;
    }

    // Method that takes ownership of `self`
    fn consume(self) -> String {
        format!("Consumed Rectangle with area: {}", self.area())
    }
}
```

---

### 2. **Calling Methods**
Methods are called using dot notation on an instance of the type. The Rust compiler automatically resolves which method to call based on the type of the instance.

#### Example: Calling Methods
```rust
fn main() {
    let mut rect = Rectangle {
        width: 10,
        height: 5,
    };

    // Call `area` method (immutable borrow)
    println!("Area: {}", rect.area());

    // Call `scale` method (mutable borrow)
    rect.scale(2);
    println!("Scaled Area: {}", rect.area());

    // Call `consume` method (takes ownership)
    let consumed = rect.consume();
    println!("{}", consumed);

    // Uncommenting the line below would cause a compile-time error because `rect` was consumed.
    // println!("Area after consume: {}", rect.area());
}
```

---

### 3. **Associated Functions**
In addition to methods, you can define **associated functions** inside an `impl` block. These are functions that are not tied to a specific instance of the type but are still associated with the type itself. They are often used for constructors or utility functions.

#### Example: Associated Function
```rust
impl Rectangle {
    // Associated function (constructor)
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
}

fn main() {
    // Call the associated function `new`
    let rect = Rectangle::new(10, 5);
    println!("Area: {}", rect.area());
}
```

---

### 4. **Key Differences Between Methods and Functions**
- **Methods** are defined within an `impl` block and operate on a specific instance of a type (via `self`).
- **Functions** are standalone and not tied to any specific type.
- Methods are called using dot notation (`instance.method()`), while functions are called directly (`function_name()`).

---

### 5. **Chaining Methods**
Rust allows method chaining when methods return `self` or a reference to `self`. This enables calling multiple methods in a single expression.

#### Example: Method Chaining
```rust
impl Rectangle {
    fn double_width(mut self) -> Self {
        self.width *= 2;
        self
    }

    fn double_height(mut self) -> Self {
        self.height *= 2;
        self
    }
}

fn main() {
    let rect = Rectangle::new(10, 5)
        .double_width()
        .double_height();

    println!("Area after doubling dimensions: {}", rect.area());
}
```

---

### 6. **Traits and Method Syntax**
Methods can also be defined in traits and implemented for types. When a trait is implemented for a type, its methods become available on instances of that type.

#### Example: Trait Implementation
```rust
trait Shape {
    fn area(&self) -> u32;
}

impl Shape for Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect = Rectangle::new(10, 5);
    println!("Area from trait: {}", rect.area());
}
```

---

### Summary
- **Methods** are defined in an `impl` block and operate on an instance of a type via `self`.
- **Associated functions** are also defined in an `impl` block but do not take `self` as a parameter.
- Methods are called using dot notation (`instance.method()`).
- Traits allow defining shared behavior across types, and their methods can be implemented for specific types.

By mastering method syntax, you can write clean, object-oriented-style code in Rust while leveraging its powerful ownership and borrowing system.
