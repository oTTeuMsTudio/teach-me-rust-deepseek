# Calculate battery power in watts
To calculate the battery power (in watts) required to match or exceed human force based on user input of mass in kilograms, we need to understand the relationship between force, work, and power. Here's a step-by-step explanation of how to approach this:

---

### **Key Concepts**
1. **Force**:
   - Force is calculated using Newton's second law: $ F = m \cdot g $, where:
     - $ F $: Force in newtons (N)
     - $ m $: Mass in kilograms (kg)
     - $ g $: Acceleration due to gravity ($ g \approx 9.81 \, \text{m/s}^2 $)

2. **Work**:
   - Work is the product of force and distance: $ W = F \cdot d $, where:
     - $ W $: Work in joules (J)
     - $ d $: Distance in meters (m)

3. **Power**:
   - Power is the rate at which work is done: $ P = \frac{W}{t} $, where:
     - $ P $: Power in watts (W)
     - $ t $: Time in seconds (s)

4. **Battery Power**:
   - To determine the battery power needed, we must know:
     - The mass ($ m $) the user provides.
     - The distance ($ d $) over which the force is applied.
     - The time ($ t $) over which the work is performed.

---

### **Program Design**
The program will:
1. Prompt the user for input:
   - Mass ($ m $) in kilograms.
   - Distance ($ d $) in meters.
   - Time ($ t $) in seconds.
2. Calculate the force ($ F $) using $ F = m \cdot g $.
3. Calculate the work ($ W $) using $ W = F \cdot d $.
4. Calculate the power ($ P $) using $ P = \frac{W}{t} $.
5. Display the required battery power in watts.

---

### **Rust Code Implementation**

Here is the Rust program that implements the above logic:

```rust
use std::io;

fn main() {
    // Constants
    const GRAVITY: f64 = 9.81; // Acceleration due to gravity in m/s^2

    // Prompt user for input
    println!("Enter the mass (in kg):");
    let mass = read_input();

    println!("Enter the distance (in meters):");
    let distance = read_input();

    println!("Enter the time (in seconds):");
    let time = read_input();

    // Ensure time is not zero to avoid division by zero
    if time == 0.0 {
        println!("Error: Time cannot be zero.");
        return;
    }

    // Calculate force, work, and power
    let force = mass * GRAVITY; // F = m * g
    let work = force * distance; // W = F * d
    let power = work / time; // P = W / t

    // Display the result
    println!(
        "To lift {} kg over {} meters in {} seconds, you need a battery with {:.2} watts of power.",
        mass, distance, time, power
    );
}

// Helper function to read user input as a floating-point number
fn read_input() -> f64 {
    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Failed to read input.");
    input.trim().parse().expect("Please enter a valid number.")
}
```

---

### **Explanation of the Code**

1. **Constants**:
   - `GRAVITY` is defined as a constant because it does not change.

2. **User Input**:
   - The `read_input` function reads a line of input from the user, trims whitespace, and parses it into a `f64`. If the input is invalid, the program will panic with an error message.

3. **Calculations**:
   - Force is calculated using $ F = m \cdot g $.
   - Work is calculated using $ W = F \cdot d $.
   - Power is calculated using $ P = \frac{W}{t} $.

4. **Error Handling**:
   - The program checks if the time is zero to avoid division by zero, which would cause a runtime error.

5. **Output**:
   - The result is displayed with two decimal places for clarity.

---

### **Example Run**

#### Input:
```
Enter the mass (in kg):
50
Enter the distance (in meters):
2
Enter the time (in seconds):
10
```

#### Output:
```
To lift 50 kg over 2 meters in 10 seconds, you need a battery with 98.10 watts of power.
```

---

### **How It Works**
1. The user inputs:
   - Mass ($ m = 50 \, \text{kg} $)
   - Distance ($ d = 2 \, \text{m} $)
   - Time ($ t = 10 \, \text{s} $)

2. Calculations:
   - Force: $ F = 50 \cdot 9.81 = 490.5 \, \text{N} $
   - Work: $ W = 490.5 \cdot 2 = 981 \, \text{J} $
   - Power: $ P = \frac{981}{10} = 98.1 \, \text{W} $

3. The program outputs the required power: $ 98.10 \, \text{W} $.

---

### **Enhancements**
1. **Input Validation**:
   - Add more robust error handling for invalid inputs (e.g., negative values).

2. **Custom Gravity**:
   - Allow the user to specify a custom value for gravity (useful for simulations on other planets).

3. **Multiple Scenarios**:
   - Allow the user to calculate power for multiple scenarios in a loop.

4. **Energy Efficiency**:
   - Factor in efficiency losses (e.g., motor inefficiency) to estimate real-world power requirements.

---

This program provides a clear and practical way to calculate the battery power needed to match human force, making it useful for educational purposes or engineering applications.
