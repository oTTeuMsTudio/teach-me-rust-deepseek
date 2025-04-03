# An Example Program Using Structs
Let's walk through an example program that calculates the area of a rectangle. We'll start with a simple implementation using individual variables, and then refactor it to use a `struct` for better organization and clarity.

---

### Step 1: Using Individual Variables
We'll begin by writing a program that uses separate variables for the width and height of the rectangle.

```rust
fn main() {
    let width = 30; // Width of the rectangle in pixels
    let height = 50; // Height of the rectangle in pixels

    println!(
        "The area of the rectangle is {} square pixels.",
        calculate_area(width, height)
    );
}

fn calculate_area(width: u32, height: u32) -> u32 {
    width * height
}
```

#### Explanation:
- We define two variables, `width` and `height`, to represent the dimensions of the rectangle.
- The `calculate_area` function takes these two variables as arguments and returns their product, which is the area of the rectangle.
- The `main` function prints the result.

While this works, it can become cumbersome if we need to pass multiple related variables around. This is where structs come in handy.

---

### Step 2: Refactoring with a Struct
Instead of using separate variables, we can group the `width` and `height` into a single `struct` called `Rectangle`.

#### Updated Code:
```rust
#[derive(Debug)] // Allows us to print the struct for debugging
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        calculate_area(&rect)
    );

    // Debugging: Print the rectangle struct
    println!("Rectangle details: {:?}", rect);
}

fn calculate_area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

#### Explanation:
1. **Defining the Struct**:
   - We define a `struct` named `Rectangle` with two fields: `width` and `height`.
   - The `#[derive(Debug)]` attribute allows us to easily print the struct for debugging purposes.

2. **Creating an Instance**:
   - In the `main` function, we create an instance of `Rectangle` with `width: 30` and `height: 50`.

3. **Passing the Struct**:
   - Instead of passing individual variables to `calculate_area`, we pass a reference to the `Rectangle` instance (`&rect`).
   - This makes the code more organized and avoids passing multiple arguments.

4. **Calculating the Area**:
   - The `calculate_area` function takes a reference to a `Rectangle` and accesses its `width` and `height` fields to compute the area.

5. **Debugging Output**:
   - The `println!` macro with `{:?}` prints the entire `Rectangle` struct, which is useful for debugging.

---

### Step 3: Adding Methods to the Struct
To make the code even more idiomatic, we can add methods to the `Rectangle` struct. This allows us to encapsulate functionality related to the struct.

#### Updated Code:
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Method to calculate the area
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // Method to check if the rectangle is a square
    fn is_square(&self) -> bool {
        self.width == self.height
    }
}

fn main() {
    let rect = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect.area()
    );

    println!(
        "Is the rectangle a square? {}",
        if rect.is_square() { "Yes" } else { "No" }
    );

    println!("Rectangle details: {:?}", rect);
}
```

#### Explanation:
1. **Adding Methods**:
   - We define an `impl` block for the `Rectangle` struct.
   - Inside the `impl` block, we add two methods:
     - `area`: Calculates and returns the area of the rectangle.
     - `is_square`: Checks if the rectangle is a square by comparing `width` and `height`.

2. **Using Methods**:
   - In the `main` function, we call `rect.area()` to calculate the area and `rect.is_square()` to check if the rectangle is a square.

3. **Improved Readability**:
   - By encapsulating functionality within the struct, the code becomes more modular and easier to understand.

---

### Final Thoughts
Using structs improves code organization and readability, especially when dealing with related data. By grouping `width` and `height` into a `Rectangle` struct and adding methods, we make the program more maintainable and extensible.

#### Final Output:
When you run the program, you should see output similar to this:
```
The area of the rectangle is 1500 square pixels.
Is the rectangle a square? No
Rectangle details: Rectangle { width: 30, height: 50 }
```

This approach demonstrates how structs and methods can be used effectively in Rust programs.
