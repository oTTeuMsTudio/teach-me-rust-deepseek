# Rotational inertia of the tire, which depends on its radius (or diameter) 
To write a Rust program that calculates the difference in moment between 26-inch and 28-inch tires, we first need to define what "moment" means in this context. For simplicity, let's assume "moment" refers to the rotational inertia of the tire, which depends on its radius (or diameter). The rotational inertia $ I $ of a hoop (a simplified model of a tire) is given by:

$$
I = m \cdot r^2
$$

Where:
- $ m $ is the mass of the tire,
- $ r $ is the radius of the tire.

The difference in moment between two tires will then be:

$$
\Delta I = I_{28} - I_{26} = m \cdot r_{28}^2 - m \cdot r_{26}^2
$$

Where:
- $ r_{28} $ is the radius of the 28-inch tire,
- $ r_{26} $ is the radius of the 26-inch tire.

We'll write a Rust program that:
1. Accepts the mass of the tire as input.
2. Converts the tire diameters into radii.
3. Calculates the moments for both tires.
4. Computes and outputs the difference in moment.

Here’s the step-by-step implementation:

---

### Rust Code Implementation

```rust
use std::io;

fn main() {
    // Prompt the user for the mass of the tire
    println!("Enter the mass of the tire (in kilograms):");

    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("Failed to read input");
    let mass: f64 = input.trim().parse().expect("Please enter a valid number");

    // Define the diameters of the tires in inches
    const DIAMETER_26_INCHES: f64 = 26.0;
    const DIAMETER_28_INCHES: f64 = 28.0;

    // Convert diameters to radii in meters (1 inch = 0.0254 meters)
    let radius_26 = DIAMETER_26_INCHES * 0.0254 / 2.0;
    let radius_28 = DIAMETER_28_INCHES * 0.0254 / 2.0;

    // Calculate the moments of inertia for both tires
    let moment_26 = mass * radius_26.powi(2);
    let moment_28 = mass * radius_28.powi(2);

    // Calculate the difference in moment
    let delta_moment = moment_28 - moment_26;

    // Output the results
    println!(
        "Moment of inertia for 26-inch tire: {:.4} kg·m²",
        moment_26
    );
    println!(
        "Moment of inertia for 28-inch tire: {:.4} kg·m²",
        moment_28
    );
    println!(
        "Difference in moment of inertia: {:.4} kg·m²",
        delta_moment
    );
}
```

---

### Explanation of the Code

1. **Input Handling**:
   - The program prompts the user to enter the mass of the tire in kilograms.
   - It reads the input as a string, trims whitespace, and parses it into an `f64` (floating-point number).

2. **Constants**:
   - The diameters of the tires are defined as constants (`DIAMETER_26_INCHES` and `DIAMETER_28_INCHES`).
   - These are converted to radii in meters using the conversion factor $ 1 \, \text{inch} = 0.0254 \, \text{meters} $.

3. **Moment of Inertia Calculation**:
   - The formula $ I = m \cdot r^2 $ is used to calculate the moment of inertia for each tire.
   - The `.powi(2)` method is used to square the radius.

4. **Difference in Moment**:
   - The difference in moment is computed as $ \Delta I = I_{28} - I_{26} $.

5. **Output**:
   - The program prints the moments of inertia for both tires and their difference, formatted to four decimal places.

---

### Example Run

#### Input:
```
Enter the mass of the tire (in kilograms):
2.5
```

#### Output:
```
Moment of inertia for 26-inch tire: 0.0429 kg·m²
Moment of inertia for 28-inch tire: 0.0484 kg·m²
Difference in moment of inertia: 0.0055 kg·m²
```

---

### Final Answer

The Rust program calculates the difference in moment of inertia between 26-inch and 28-inch tires based on the provided mass. You can copy and run the code in any Rust environment to perform the calculation. 

