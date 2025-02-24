# Convert temperatures between Fahrenheit and Celsius

Converting temperatures between Fahrenheit and Celsius is a common task in programming. Below, I will guide you through implementing this functionality in Rust step by step. We'll create functions to convert from Fahrenheit to Celsius and vice versa.

---

### **Understanding the Conversion Formulas**
1. **Fahrenheit to Celsius**:  
   \[
   C = \frac{5}{9} \times (F - 32)
   \]
   Where \( F \) is the temperature in Fahrenheit and \( C \) is the temperature in Celsius.

2. **Celsius to Fahrenheit**:  
   \[
   F = \left( \frac{9}{5} \times C \right) + 32
   \]
   Where \( C \) is the temperature in Celsius and \( F \) is the temperature in Fahrenheit.

---

### **Step-by-Step Implementation in Rust**

#### 1. Define the Conversion Functions
We'll write two functions:
- `fahrenheit_to_celsius`: Converts Fahrenheit to Celsius.
- `celsius_to_fahrenheit`: Converts Celsius to Fahrenheit.

Here's the code:

```rust
// Function to convert Fahrenheit to Celsius
fn fahrenheit_to_celsius(f: f64) -> f64 {
    (f - 32.0) * 5.0 / 9.0
}

// Function to convert Celsius to Fahrenheit
fn celsius_to_fahrenheit(c: f64) -> f64 {
    (c * 9.0 / 5.0) + 32.0
}
```

#### 2. Test the Functions
To ensure the functions work correctly, we can test them with some example values.

```rust
fn main() {
    // Example temperatures
    let temp_fahrenheit = 98.6; // Body temperature in Fahrenheit
    let temp_celsius = 37.0;    // Body temperature in Celsius

    // Convert Fahrenheit to Celsius
    let converted_celsius = fahrenheit_to_celsius(temp_fahrenheit);
    println!("{}°F is equivalent to {:.2}°C", temp_fahrenheit, converted_celsius);

    // Convert Celsius to Fahrenheit
    let converted_fahrenheit = celsius_to_fahrenheit(temp_celsius);
    println!("{}°C is equivalent to {:.2}°F", temp_celsius, converted_fahrenheit);
}
```

#### 3. Explanation of the Code
- **Data Type**: We use `f64` for floating-point numbers because temperature values often involve decimals.
- **Mathematical Operations**: The formulas are implemented directly as arithmetic operations.
- **Formatting Output**: The `println!` macro uses `{:.2}` to format the output to two decimal places for better readability.

---

### **Complete Code**
Here’s the full program combining everything:

```rust
// Function to convert Fahrenheit to Celsius
fn fahrenheit_to_celsius(f: f64) -> f64 {
    (f - 32.0) * 5.0 / 9.0
}

// Function to convert Celsius to Fahrenheit
fn celsius_to_fahrenheit(c: f64) -> f64 {
    (c * 9.0 / 5.0) + 32.0
}

fn main() {
    // Example temperatures
    let temp_fahrenheit = 98.6; // Body temperature in Fahrenheit
    let temp_celsius = 37.0;    // Body temperature in Celsius

    // Convert Fahrenheit to Celsius
    let converted_celsius = fahrenheit_to_celsius(temp_fahrenheit);
    println!("{}°F is equivalent to {:.2}°C", temp_fahrenheit, converted_celsius);

    // Convert Celsius to Fahrenheit
    let converted_fahrenheit = celsius_to_fahrenheit(temp_celsius);
    println!("{}°C is equivalent to {:.2}°F", temp_celsius, converted_fahrenheit);
}
```

---

### **Output**
When you run the program, you should see the following output:

```
98.6°F is equivalent to 37.00°C
37.0°C is equivalent to 98.60°F
```

---

### **Extending the Program**
You can extend this program by:
1. Allowing user input for the temperature values.
2. Adding error handling for invalid inputs.
3. Creating a loop to allow multiple conversions without restarting the program.

Let me know if you'd like help with any of these extensions!
