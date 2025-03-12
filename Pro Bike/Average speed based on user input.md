# a Rust program that calculates average speed based on user input

```rust
use std::io;

fn main() {
    let distance = get_valid_number("Enter the distance traveled (in kilometers):");
    let time = get_valid_number("Enter the time taken (in hours):");

    while time == 0.0 {
        println!("Time cannot be zero. Please enter a non-zero value.");
        let new_time = get_valid_number("Enter the time taken (in hours):");
        if new_time != 0.0 {
            break;
        }
    }

    let average_speed = distance / time;
    println!("The average speed is {:.2} km/h", average_speed);
}

fn get_valid_number(prompt: &str) -> f64 {
    loop {
        println!("{}", prompt);
        let mut input = String::new();
        io::stdin()
            .read_line(&mut input)
            .expect("Failed to read input");

        match input.trim().parse::<f64>() {
            Ok(num) => return num,
            Err(_) => {
                println!("Invalid input. Please enter a valid number.");
                continue;
            }
        }
    }
}
```

**Explanation:**

1. **Input Validation:**
   - The `get_valid_number` function repeatedly prompts the user until a valid number is entered.
   - It trims whitespace and attempts to parse the input as an `f64` floating-point number.

2. **Zero Division Handling:**
   - After getting the initial time input, we check if it's zero.
   - If zero is entered, the program enters a loop that continues prompting until a non-zero time is provided.

3. **Calculation and Output:**
   - The average speed is calculated as `distance / time`.
   - The result is formatted to two decimal places using `{:.2}` in the print statement.

**Example Usage:**
```
Enter the distance traveled (in kilometers):
150
Enter the time taken (in hours):
2.5
The average speed is 60.00 km/h
```

This program ensures:
- Robust input handling for non-numeric values
- Protection against division by zero
- Clear user feedback for invalid inputs
- Precise output formatting with two decimal places
