# Calculate seat height (practice programming while solving a real-world problem)

To write a program in Rust that calculates the height of the seat on a professional road bike, we need to consider some basic principles. The height of the bike seat is typically determined by the rider's inseam (the length from the ground to the crotch) and a multiplier factor. A common formula for calculating the seat height is:

$$
\text{Seat Height} = \text{Inseam} \times \text{Multiplier}
$$

The multiplier is often around **0.883** for road bikes, but it can vary slightly depending on personal preference or specific bike geometry.

### Steps to Write the Program:
1. **Input**: Ask the user to input their inseam measurement.
2. **Calculation**: Use the formula to calculate the seat height.
3. **Output**: Display the calculated seat height to the user.

Below is the Rust implementation of this program:

---

### Rust Code: Bike Seat Height Calculator

```rust
use std::io;

fn main() {
    // Prompt the user to enter their inseam measurement
    println!("Enter your inseam measurement in centimeters:");

    // Create a mutable string to store user input
    let mut inseam_input = String::new();

    // Read the user input from the console
    io::stdin()
        .read_line(&mut inseam_input)
        .expect("Failed to read input");

    // Parse the input into a floating-point number
    let inseam: f64 = match inseam_input.trim().parse() {
        Ok(num) => num,
        Err(_) => {
            println!("Please enter a valid number.");
            return;
        }
    };

    // Define the multiplier for road bikes
    const MULTIPLIER: f64 = 0.883;

    // Calculate the seat height using the formula
    let seat_height = inseam * MULTIPLIER;

    // Output the result to the user
    println!(
        "Your recommended seat height is {:.2} cm.",
        seat_height
    );
}
```

---

### Explanation of the Code:
1. **Input Handling**:
   - The program prompts the user to enter their inseam measurement in centimeters.
   - The `io::stdin().read_line()` function reads the input as a string.
   - The `.trim().parse()` method converts the string into a floating-point number (`f64`). If the input is invalid, the program exits with an error message.

2. **Multiplier**:
   - The constant `MULTIPLIER` is set to `0.883`, which is a standard value for road bikes. You can adjust this value if needed.

3. **Calculation**:
   - The seat height is calculated using the formula: `seat_height = inseam * MULTIPLIER`.

4. **Output**:
   - The result is displayed with two decimal places using the format specifier `{:.2}`.

---

### Example Run:
#### Input:
```
Enter your inseam measurement in centimeters:
80
```

#### Output:
```
Your recommended seat height is 70.64 cm.
```

---

### Additional Features (Optional):
If you want to make the program more robust or user-friendly, you could:
1. **Validate Input Range**:
   - Ensure the inseam is within a reasonable range (e.g., 50–120 cm).
   - Add a check to reject unrealistic values.

2. **Support Multiple Units**:
   - Allow the user to input measurements in inches or centimeters and convert them accordingly.

3. **Custom Multiplier**:
   - Let the user specify their preferred multiplier if they have a custom setup.

Here’s an example of how you might validate the input range:

```rust
if inseam < 50.0 || inseam > 120.0 {
    println!("Please enter an inseam measurement between 50 and 120 cm.");
    return;
}
```

---

### Final Answer:
The Rust program above calculates the seat height of a road bike based on the user's inseam measurement. It uses a standard multiplier of `0.883` and provides accurate results with two decimal places. You can extend the program further to include additional features like input validation or unit conversion.
