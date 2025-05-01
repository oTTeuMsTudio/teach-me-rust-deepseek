To solve the problem of computing the **median** and **mode** of a list of integers in Rust, we utilize two key data structures:

- A **vector** to store and sort the input list for computing the median.
- A **hash map** to count the frequency of each integer for computing the mode.

The solution follows a clear and structured approach:

---

### 🧮 Median Calculation

The **median** is the middle value when the list is sorted. If the list has an **odd** number of elements, the median is the middle element. If the list has an **even** number of elements, the median is the average of the two middle elements.

Steps:
1. Clone and sort the input vector.
2. Determine the length of the sorted vector.
3. If the length is even, compute the average of the two middle elements.
4. If the length is odd, return the middle element.

---

### 📊 Mode Calculation

The **mode** is the value that occurs most frequently in the list. To compute this efficiently:

Steps:
1. Use a `HashMap` to count the frequency of each integer.
2. While iterating through the list, keep track of the current maximum frequency (`max_count`) and the corresponding number (`mode`).
3. If a new frequency exceeds the current `max_count`, update `max_count` and `mode`.

This ensures that the mode is the **first** element that reaches the highest frequency, which provides a deterministic result even in the case of ties.

---

### ✅ Full Implementation in Rust

```rust
use std::collections::HashMap;

/// Computes the median and mode of a list of integers.
///
/// # Arguments
/// * `nums` - A slice of integers.
///
/// # Returns
/// A tuple `(median: f64, mode: i32)`.
///
/// # Panics
/// If the input slice is empty.
pub fn median_and_mode(nums: &[i32]) -> (f64, i32) {
    if nums.is_empty() {
        panic!("Input list cannot be empty.");
    }

    // Step 1: Compute the median
    let mut sorted = nums.to_vec();
    sorted.sort();
    let len = sorted.len();

    let median = if len % 2 == 0 {
        let mid = len / 2;
        (sorted[mid - 1] as f64 + sorted[mid] as f64) / 2.0
    } else {
        sorted[len / 2] as f64
    };

    // Step 2: Compute the mode
    let mut counts = HashMap::new();
    let mut max_count = 0;
    let mut mode = nums[0];

    for &num in nums {
        let count = counts.entry(num).and_modify(|e| *e += 1).or_insert(1);
        if *count > max_count {
            max_count = *count;
            mode = num;
        }
    }

    (median, mode)
}
```

---

### 🧪 Example Usage

```rust
fn main() {
    let numbers = vec![3, 1, 2, 3, 4, 2, 3];
    let (median, mode) = median_and_mode(&numbers);
    println!("Median: {}", median); // Output: 3.0
    println!("Mode: {}", mode);     // Output: 3
}
```

---

### 📌 Notes

- The function assumes the input list is **non-empty**. If it's empty, it will panic.
- The **mode** is determined by the first element that reaches the highest frequency, ensuring deterministic behavior in case of ties.
- The **median** is returned as an `f64` to accommodate averages of even-length lists.

---

This implementation is efficient, idiomatic, and adheres to Rust best practices while leveraging the power of vectors and hash maps.
