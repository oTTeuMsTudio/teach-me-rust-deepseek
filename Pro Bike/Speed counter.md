# Create speed counter
To create a program that calculates the speed of a road bike based on input from a wheel sensor, we need to consider the following steps:

1. **Understand the physics**: The speed of the bike can be calculated using the formula:
   $$
   \text{Speed} = \frac{\text{Distance per revolution} \times \text{Revolutions per second}}{\text{Time}}
   $$
   Here:
   - Distance per revolution is the circumference of the wheel, which depends on the wheel's diameter.
   - Revolutions per second are derived from the sensor data (e.g., how often the sensor detects a full rotation).

2. **Inputs**:
   - Wheel diameter in inches (user-provided).
   - Time between sensor activations (simulated or provided as input).

3. **Outputs**:
   - Speed in miles per hour (or another unit, if desired).

4. **Implementation in Rust**:
   - We'll use standard input for user interaction.
   - Convert the wheel diameter to circumference using the formula $ C = \pi \times d $, where $ d $ is the diameter.
   - Simulate sensor input or allow the user to input time intervals between rotations.
   - Calculate and display the speed.

Here’s the complete Rust program:

```rust
use std::io::{self, Write};
use std::f64::consts::PI;

fn main() {
    // Step 1: Get user input for wheel diameter in inches
    let wheel_diameter = get_user_input("Enter the wheel diameter in inches: ");
    
    // Step 2: Convert diameter to circumference (in inches)
    let circumference_in_inches = PI * wheel_diameter;
    let circumference_in_miles = circumference_in_inches / (12.0 * 5280.0); // Convert inches to miles

    println!("Wheel circumference: {:.2} inches ({:.6} miles)", circumference_in_inches, circumference_in_miles);

    loop {
        // Step 3: Simulate sensor input (time between rotations in seconds)
        let time_between_rotations = get_user_input("Enter time between rotations in seconds (or 0 to exit): ");

        if time_between_rotations == 0.0 {
            println!("Exiting...");
            break;
        }

        // Step 4: Calculate speed in miles per hour
        let revolutions_per_second = 1.0 / time_between_rotations;
        let speed_mph = revolutions_per_second * circumference_in_miles * 3600.0; // Convert to mph

        // Step 5: Display the result
        println!("Speed: {:.2} mph", speed_mph);
    }
}

// Helper function to get user input as a floating-point number
fn get_user_input(prompt: &str) -> f64 {
    loop {
        print!("{}", prompt);
        io::stdout().flush().unwrap(); // Ensure the prompt is displayed immediately

        let mut input = String::new();
        io::stdin().read_line(&mut input).expect("Failed to read input");

        match input.trim().parse::<f64>() {
            Ok(num) => return num,
            Err(_) => println!("Invalid input. Please enter a valid number."),
        }
    }
}
```

---

### Explanation of the Code:

1. **User Input**:
   - The `get_user_input` function prompts the user for input and ensures it is parsed as a valid floating-point number (`f64`). If the input is invalid, it loops until a valid number is entered.

2. **Wheel Circumference**:
   - The circumference is calculated using $ C = \pi \times d $. Since the input is in inches, we convert it to miles by dividing by $ 12 \times 5280 $ (inches per mile).

3. **Speed Calculation**:
   - The speed is calculated using the formula:
     $$
     \text{Speed (mph)} = \left(\frac{1}{\text{Time between rotations (seconds)}}\right) \times \text{Circumference (miles)} \times 3600
     $$
   - The factor of 3600 converts seconds to hours.

4. **Looping for Multiple Inputs**:
   - The program runs in a loop, allowing the user to input multiple time intervals. Entering `0` exits the loop.

5. **Output**:
   - The speed is displayed in miles per hour with two decimal places.

---

### Example Interaction:

```
Enter the wheel diameter in inches: 27
Wheel circumference: 84.82 inches (0.001340 miles)
Enter time between rotations in seconds (or 0 to exit): 1
Speed: 4.82 mph
Enter time between rotations in seconds (or 0 to exit): 0.5
Speed: 9.64 mph
Enter time between rotations in seconds (or 0 to exit): 0
Exiting...
```

---

### Key Points:
- The program handles invalid input gracefully.
- It uses standard physics formulas to calculate speed.
- The user can simulate different scenarios by entering various time intervals.

This implementation provides a robust and interactive way to calculate bike speed based on sensor data.
