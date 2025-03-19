# Simulater counter that calculates rotations
Writing a program to simulate a counter that calculates rotations from a sensor on a road bike in Rust involves several steps. We'll break it down into manageable parts:

1. **Understand the Problem**: A sensor on the wheel detects each rotation of the wheel, and we need to count these rotations over time. The program should simulate this behavior.

2. **Program Design**:
   - Simulate the sensor input (e.g., detecting wheel rotations).
   - Maintain a counter to track the number of rotations.
   - Optionally, calculate additional metrics like speed or distance traveled based on the wheel's circumference.

3. **Implementation**:
   - Use a loop to simulate sensor inputs over time.
   - Increment the rotation counter when a "rotation" is detected.
   - Output the results periodically or at the end of the simulation.

Below is a step-by-step implementation in Rust:

---

### Rust Code: Simulating a Bike Rotation Counter

```rust
use std::thread;
use std::time::{Duration, Instant};

fn main() {
    // Constants for simulation
    const WHEEL_CIRCUMFERENCE: f64 = 2.1; // Wheel circumference in meters (example: 700c wheel)
    const SIMULATION_DURATION: u64 = 10; // Duration of simulation in seconds
    const ROTATION_INTERVAL: u64 = 500; // Time between simulated rotations in milliseconds

    // Variables to track state
    let mut rotation_count = 0;
    let start_time = Instant::now();

    println!("Starting bike rotation simulation...");

    // Simulate sensor input and count rotations
    while start_time.elapsed().as_secs() < SIMULATION_DURATION {
        // Simulate a rotation detected by the sensor
        thread::sleep(Duration::from_millis(ROTATION_INTERVAL));
        rotation_count += 1;

        // Print current count periodically
        println!(
            "Rotation detected! Total rotations: {}, Elapsed time: {:.2}s",
            rotation_count,
            start_time.elapsed().as_secs_f64()
        );
    }

    // Calculate total distance traveled
    let total_distance = rotation_count as f64 * WHEEL_CIRCUMFERENCE;
    let total_time = start_time.elapsed().as_secs_f64();
    let average_speed = if total_time > 0.0 {
        total_distance / total_time
    } else {
        0.0
    };

    // Print final results
    println!("\nSimulation complete!");
    println!("Total rotations: {}", rotation_count);
    println!("Total distance traveled: {:.2} meters", total_distance);
    println!("Average speed: {:.2} m/s", average_speed);
}
```

---

### Explanation of the Code

1. **Constants**:
   - `WHEEL_CIRCUMFERENCE`: Represents the circumference of the bike wheel in meters. You can adjust this based on your bike's wheel size.
   - `SIMULATION_DURATION`: The duration of the simulation in seconds.
   - `ROTATION_INTERVAL`: The time interval between simulated wheel rotations in milliseconds.

2. **State Variables**:
   - `rotation_count`: Tracks the number of rotations detected.
   - `start_time`: Records the start time of the simulation using `Instant`.

3. **Simulation Loop**:
   - The `while` loop runs for the duration of the simulation (`SIMULATION_DURATION`).
   - Inside the loop, the program simulates a rotation by sleeping for `ROTATION_INTERVAL` milliseconds and then increments the `rotation_count`.
   - Periodic updates are printed to show the current rotation count and elapsed time.

4. **Final Calculations**:
   - After the simulation ends, the program calculates:
     - Total distance traveled (`rotation_count * WHEEL_CIRCUMFERENCE`).
     - Average speed (`total_distance / total_time`).

5. **Output**:
   - The program prints the total rotations, distance traveled, and average speed at the end of the simulation.

---

### Example Output

When you run the program, you might see output like this:

```
Starting bike rotation simulation...
Rotation detected! Total rotations: 1, Elapsed time: 0.50s
Rotation detected! Total rotations: 2, Elapsed time: 1.00s
Rotation detected! Total rotations: 3, Elapsed time: 1.50s
...
Rotation detected! Total rotations: 20, Elapsed time: 10.00s

Simulation complete!
Total rotations: 20
Total distance traveled: 42.00 meters
Average speed: 4.20 m/s
```

---

### Customization and Extensions

1. **Real Sensor Input**:
   - Replace the simulated `thread::sleep` with actual sensor input (e.g., reading from a GPIO pin or serial connection).

2. **Dynamic Speed**:
   - Instead of a fixed `ROTATION_INTERVAL`, vary the interval to simulate changes in speed.

3. **Distance Units**:
   - Convert the distance to kilometers or miles if needed.

4. **GUI or Logging**:
   - Add a graphical interface or log the data to a file for further analysis.

---

This program provides a foundation for simulating a bike rotation counter in Rust. You can expand it to include more advanced features as needed. Let me know if you have further questions!
