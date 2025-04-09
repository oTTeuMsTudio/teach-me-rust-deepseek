# Enums and Pattern Matching
Let's break down **enums** and **pattern matching** in Rust step by step. I'll explain these concepts in a simple, beginner-friendly way.

---

### **1. What are Enums?**

Enums (short for "enumerations") are a way to define a type that can have a fixed set of values. Think of them as a list of possible options or states.

For example:
- A `Direction` enum could represent the four cardinal directions: `North`, `South`, `East`, and `West`.
- A `Result` enum could represent success (`Ok`) or failure (`Err`).

Here’s how you define an enum in Rust:

```rust
enum Direction {
    North,
    South,
    East,
    West,
}
```

This means `Direction` can only be one of these four values: `North`, `South`, `East`, or `West`.

---

### **2. Why Use Enums?**

Enums are useful when you want to represent a value that has a limited number of possibilities. For example:
- Days of the week (`Monday`, `Tuesday`, etc.)
- States of a machine (`Idle`, `Running`, `Stopped`)
- Responses from a function (`Success`, `Failure`)

---

### **3. Enum Variants Can Hold Data**

Enums can also hold additional data in their variants. For example:

```rust
enum Message {
    Quit,                       // No data
    Move { x: i32, y: i32 },    // Struct-like data
    Write(String),              // Tuple-like data
    ChangeColor(i32, i32, i32), // Tuple-like data
}
```

Here:
- `Quit` is just a simple variant with no data.
- `Move` holds two integers (`x` and `y`).
- `Write` holds a `String`.
- `ChangeColor` holds three integers (RGB values).

This makes enums very flexible!

---

### **4. Pattern Matching with `match`**

Pattern matching is a powerful feature in Rust that lets you handle different cases of an enum. The `match` keyword is used for this.

Here’s an example:

```rust
enum Direction {
    North,
    South,
    East,
    West,
}

fn which_way_to_go(dir: Direction) {
    match dir {
        Direction::North => println!("Heading North!"),
        Direction::South => println!("Heading South!"),
        Direction::East => println!("Heading East!"),
        Direction::West => println!("Heading West!"),
    }
}
```

In this example:
- The `match` statement checks the value of `dir`.
- Depending on whether it’s `North`, `South`, `East`, or `West`, it executes the corresponding code.

---

### **5. Handling Enums with Data**

If your enum holds data, you can extract and use that data in the `match` statement. Here’s an example with the `Message` enum we defined earlier:

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn process_message(msg: Message) {
    match msg {
        Message::Quit => println!("Quit message received."),
        Message::Move { x, y } => println!("Move to ({}, {})", x, y),
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => println!("Change color to RGB({}, {}, {})", r, g, b),
    }
}
```

Here:
- For `Move`, we extract `x` and `y` and print them.
- For `Write`, we extract the `text` and print it.
- For `ChangeColor`, we extract the RGB values and print them.

---

### **6. Using `_` for Default Cases**

If you don’t care about some cases, you can use `_` as a wildcard:

```rust
fn check_direction(dir: Direction) {
    match dir {
        Direction::North => println!("Going North!"),
        _ => println!("Going somewhere else..."),
    }
}
```

The `_` matches any value not explicitly listed. This is useful when you don’t need to handle every case.

---

### **7. Returning Values from `match`**

You can also return values from a `match` statement. For example:

```rust
fn is_north(dir: Direction) -> bool {
    match dir {
        Direction::North => true,
        _ => false,
    }
}
```

Here, the `match` returns `true` if the direction is `North`, otherwise `false`.

---

### **8. Combining Enums with `Option` and `Result`**

Rust has two common enums built into the language:
- `Option<T>`: Represents either `Some(value)` or `None`. It’s used when a value might be absent.
- `Result<T, E>`: Represents either `Ok(value)` or `Err(error)`. It’s used for operations that might fail.

Example with `Option`:

```rust
fn divide(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None // Division by zero is undefined
    } else {
        Some(a / b)
    }
}

fn main() {
    let result = divide(10.0, 2.0);
    match result {
        Some(value) => println!("Result: {}", value),
        None => println!("Cannot divide by zero!"),
    }
}
```

---

### **9. Concise Pattern Matching with `if let`**

For simple cases where you only care about one variant, you can use `if let` instead of `match`:

```rust
let msg = Message::Write(String::from("Hello"));

if let Message::Write(text) = msg {
    println!("Text message: {}", text);
} else {
    println!("Not a Write message.");
}
```

This is shorter than writing a full `match` when you only care about one case.

---

### **Summary**

- **Enums** let you define a type with a fixed set of possible values.
- You can attach data to enum variants, making them very flexible.
- **Pattern matching** with `match` allows you to handle each variant of an enum.
- Use `_` for default cases and `if let` for concise handling of specific cases.

---

### **Practice Exercise**

Define an enum called `Shape` with variants `Circle(f64)`, `Rectangle(f64, f64)`, and `Triangle(f64, f64, f64)`. Write a function `calculate_area` that takes a `Shape` and calculates its area using pattern matching.

Here’s a hint:

```rust
enum Shape {
    Circle(f64),
    Rectangle(f64, f64),
    Triangle(f64, f64, f64),
}

fn calculate_area(shape: Shape) -> f64 {
    match shape {
        Shape::Circle(radius) => std::f64::consts::PI * radius * radius,
        Shape::Rectangle(width, height) => width * height,
        Shape::Triangle(a, b, c) => {
            let s = (a + b + c) / 2.0;
            (s * (s - a) * (s - b) * (s - c)).sqrt()
        }
    }
}
