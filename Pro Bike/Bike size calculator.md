# Calculate bike size (practice programming while solving a real-world problem)

Bike size calculation depends on several factors, such as the rider's height, inseam length, and sometimes their riding style or preferences. For simplicity, we'll create a basic program that calculates bike frame size based on the rider's height.

---

### Problem Breakdown
1. **Input**: Rider's height (e.g., in centimeters or inches).
2. **Logic**: Use a formula to calculate the appropriate bike frame size. A common formula for road bikes is:
   - Frame size (in cm) = Height (in cm) × 0.67
3. **Output**: Display the calculated bike frame size.

We'll write a simple command-line program in Rust to achieve this.

---

### Step-by-Step Implementation

#### 1. Setting Up the Project
First, ensure you have Rust installed. If not, install it from [rust-lang.org](https://www.rust-lang.org/).

Create a new Rust project:
```bash
cargo new bike_size_calculator
cd bike_size_calculator
```

#### 2. Writing the Code
Open the `src/main.rs` file and replace its contents with the following code:

```rust
use std::io;

fn main() {
    println!("Welcome to the Bike Size Calculator!");

    // Prompt the user for their height
    println!("Please enter your height in centimeters:");

    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Failed to read input");

    // Parse the input into a floating-point number
    let height: f32 = match input.trim().parse() {
        Ok(num) => num,
        Err(_) => {
            println!("Invalid input. Please enter a valid number.");
            return;
        }
    };

    // Calculate the bike frame size using the formula
    let frame_size = height * 0.67;

    // Display the result
    println!(
        "Based on your height of {:.2} cm, your recommended bike frame size is {:.2} cm.",
        height, frame_size
    );
}
```

---

#### 3. Explanation of the Code
1. **User Input**:
   - We use `std::io` to read input from the user.
   - The `read_line` function captures the input as a string.

2. **Parsing Input**:
   - The `trim()` method removes any extra whitespace or newline characters.
   - The `parse()` function converts the string into a floating-point number (`f32`).
   - If parsing fails, we handle the error gracefully by printing an error message and exiting the program.

3. **Calculation**:
   - The bike frame size is calculated using the formula: `frame_size = height * 0.67`.

4. **Output**:
   - The result is displayed using `println!`, formatted to two decimal places for clarity.

---

#### 4. Running the Program
To run the program, use the following command:
```bash
cargo run
```

You'll see output like this:
```
Welcome to the Bike Size Calculator!
Please enter your height in centimeters:
170
Based on your height of 170.00 cm, your recommended bike frame size is 113.90 cm.
```

---

### Enhancements
Here are some ideas to make the program more robust and user-friendly:
1. **Support for Inches**:
   - Allow users to input their height in inches and convert it to centimeters (1 inch = 2.54 cm).
2. **Different Bike Types**:
   - Add options for different bike types (e.g., road, mountain, hybrid) with varying formulas.
3. **Error Handling**:
   - Validate the input range (e.g., ensure height is within reasonable limits).
4. **Graphical Interface**:
   - Use a GUI library like `egui` or `tauri` for a more interactive experience.

---

### Example Enhancement: Supporting Inches
Here’s how you can modify the program to support height in inches:

```rust
use std::io;

fn main() {
    println!("Welcome to the Bike Size Calculator!");
    println!("Do you want to enter your height in [1] centimeters or [2] inches?");

    let mut choice = String::new();
    io::stdin()
        .read_line(&mut choice)
        .expect("Failed to read input");

    let height_cm = match choice.trim() {
        "1" => {
            println!("Enter your height in centimeters:");
            let mut input = String::new();
            io::stdin()
                .read_line(&mut input)
                .expect("Failed to read input");
            input.trim().parse::<f32>().expect("Invalid input")
        }
        "2" => {
            println!("Enter your height in inches:");
            let mut input = String::new();
            io::stdin()
                .read_line(&mut input)
                .expect("Failed to read input");
            let height_in = input.trim().parse::<f32>().expect("Invalid input");
            height_in * 2.54 // Convert inches to centimeters
        }
        _ => {
            println!("Invalid choice. Please restart the program.");
            return;
        }
    };

    let frame_size = height_cm * 0.67;
    println!(
        "Based on your height, your recommended bike frame size is {:.2} cm.",
        frame_size
    );
}
```

---

### Final Thoughts
This program demonstrates how to take user input, perform calculations, and display results in Rust. You can expand it further based on your needs, such as adding more customization options or integrating it into a larger application.
