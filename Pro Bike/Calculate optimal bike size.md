# Calculate Optimal Road Bike Size

Lets go through the process of creating a simple Rust program that calculates the optimal road bike size (S, M, L, XL) based on the user's body height. Here's how we can approach this:

---

### **Step 1: Understand the Problem**
The goal is to determine the optimal road bike size based on the user's body height. Typically, bike sizes are categorized as follows:
- **Small (S):** For users with a body height between 150 cm and 165 cm.
- **Medium (M):** For users with a body height between 166 cm and 175 cm.
- **Large (L):** For users with a body height between 176 cm and 185 cm.
- **Extra Large (XL):** For users with a body height above 185 cm.

We'll use these ranges to map the user's input to the appropriate bike size.

---

### **Step 2: Plan the Program**
1. **Input:** Prompt the user to enter their body height in centimeters.
2. **Processing:** Use conditional logic (`if`, `else if`) to determine the bike size based on the input.
3. **Output:** Display the recommended bike size to the user.

---

### **Step 3: Write the Code**
Here’s the complete Rust program:

```rust
use std::io;

fn main() {
    // Prompt the user for their body height
    println!("Enter your body height in centimeters:");

    // Create a mutable string to store the user's input
    let mut input = String::new();

    // Read the user's input from the console
    io::stdin()
        .read_line(&mut input)
        .expect("Failed to read input");

    // Convert the input string to an integer
    let height: i32 = match input.trim().parse() {
        Ok(num) => num,
        Err(_) => {
            println!("Please enter a valid number.");
            return;
        }
    };

    // Determine the optimal bike size based on the height
    let bike_size = if height >= 150 && height <= 165 {
        "Small (S)"
    } else if height >= 166 && height <= 175 {
        "Medium (M)"
    } else if height >= 176 && height <= 185 {
        "Large (L)"
    } else if height > 185 {
        "Extra Large (XL)"
    } else {
        "No recommendation available for heights below 150 cm."
    };

    // Output the result
    println!("Based on your height of {} cm, the recommended bike size is: {}", height, bike_size);
}
```

---

### **Step 4: Explanation of the Code**
1. **Input Handling:**
   - We use `std::io` to handle user input.
   - The `read_line` function reads the input as a string, and `trim()` removes any extra whitespace or newline characters.
   - The `parse()` method converts the string into an integer (`i32`). If the conversion fails, we print an error message and exit.

2. **Logic for Bike Size Calculation:**
   - The program uses a series of `if`, `else if`, and `else` statements to check the user's height against predefined ranges.
   - Based on the range, it assigns the appropriate bike size.

3. **Output:**
   - The program prints the recommended bike size along with the user's height.

---

### **Step 5: Test the Program**
Let’s test the program with a few inputs:
- **Input:** `190`
  - **Output:** `Based on your height of 190 cm, the recommended bike size is: Extra Large (XL)`
- **Input:** `170`
  - **Output:** `Based on your height of 170 cm, the recommended bike size is: Medium (M)`
- **Input:** `140`
  - **Output:** `Based on your height of 140 cm, the recommended bike size is: No recommendation available for heights below 150 cm.`

---

### **Step 6: Enhancements**
If you want to make the program more robust or user-friendly, consider:
1. Adding input validation to ensure the user enters a positive number.
2. Allowing the user to input height in other units (e.g., feet and inches) and converting it to centimeters.
3. Using a `match` statement instead of `if`/`else if` for cleaner code.

---

### **Final Answer**
Here’s the boxed final version of the program:

```rust
use std::io;

fn main() {
    println!("Enter your body height in centimeters:");
    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Failed to read input");
    let height: i32 = match input.trim().parse() {
        Ok(num) => num,
        Err(_) => {
            println!("Please enter a valid number.");
            return;
        }
    };
    let bike_size = if height >= 150 && height <= 165 {
        "Small (S)"
    } else if height >= 166 && height <= 175 {
        "Medium (M)"
    } else if height >= 176 && height <= 185 {
        "Large (L)"
    } else if height > 185 {
        "Extra Large (XL)"
    } else {
        "No recommendation available for heights below 150 cm."
    };
    println!("Based on your height of {} cm, the recommended bike size is: {}", height, bike_size);
}
```

This program is simple, functional, and adheres to Rust's best practices.
