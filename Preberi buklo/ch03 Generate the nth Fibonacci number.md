# Generate the nth Fibonacci number

To generate the nth Fibonacci number in Rust, we can use either an iterative or recursive approach. Below, I will explain both methods and provide code examples for each.

---

### **1. Iterative Approach**
The iterative approach is efficient and avoids the overhead of recursion. It uses a loop to calculate the Fibonacci sequence up to the nth number.

#### **Explanation:**
- Start with two initial values: `0` and `1`.
- Use a loop to calculate the next Fibonacci number by summing the previous two numbers.
- Continue this process until you reach the nth number.

#### **Code Example:**
```rust
fn fibonacci(n: u32) -> u64 {
    if n == 0 {
        return 0;
    } else if n == 1 {
        return 1;
    }

    let mut a: u64 = 0; // First Fibonacci number
    let mut b: u64 = 1; // Second Fibonacci number

    for _ in 2..=n {
        let temp = a + b; // Calculate the next Fibonacci number
        a = b;           // Update the first number
        b = temp;        // Update the second number
    }

    b // Return the nth Fibonacci number
}

fn main() {
    let n = 10; // Example: Find the 10th Fibonacci number
    println!("The {}th Fibonacci number is {}", n, fibonacci(n));
}
```

#### **Output:**
```
The 10th Fibonacci number is 55
```

---

### **2. Recursive Approach**
The recursive approach directly follows the mathematical definition of the Fibonacci sequence:
\[ F(0) = 0, \quad F(1) = 1, \quad F(n) = F(n-1) + F(n-2) \]

#### **Explanation:**
- Define a function that calls itself to compute the Fibonacci numbers.
- This method is simple but inefficient for large `n` because it recalculates the same values multiple times.

#### **Code Example:**
```rust
fn fibonacci_recursive(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2),
    }
}

fn main() {
    let n = 10; // Example: Find the 10th Fibonacci number
    println!("The {}th Fibonacci number is {}", n, fibonacci_recursive(n));
}
```

#### **Output:**
```
The 10th Fibonacci number is 55
```

#### **Note:**
The recursive approach is not recommended for large `n` (e.g., `n > 30`) because its time complexity is exponential \(O(2^n)\). For better performance, consider using memoization or the iterative approach.

---

### **3. Optimized Recursive Approach with Memoization**
Memoization stores previously computed results to avoid redundant calculations, making the recursive approach more efficient.

#### **Code Example:**
```rust
use std::collections::HashMap;

fn fibonacci_memo(n: u32, memo: &mut HashMap<u32, u64>) -> u64 {
    if let Some(&value) = memo.get(&n) {
        return value;
    }

    let result = if n == 0 {
        0
    } else if n == 1 {
        1
    } else {
        fibonacci_memo(n - 1, memo) + fibonacci_memo(n - 2, memo)
    };

    memo.insert(n, result);
    result
}

fn main() {
    let n = 10; // Example: Find the 10th Fibonacci number
    let mut memo = HashMap::new();
    println!("The {}th Fibonacci number is {}", n, fibonacci_memo(n, &mut memo));
}
```

#### **Output:**
```
The 10th Fibonacci number is 55
```

#### **Advantages:**
- Time complexity is reduced to \(O(n)\).
- Space complexity is \(O(n)\) due to the hash map used for memoization.

---

### **Conclusion**
- Use the **iterative approach** for simplicity and efficiency.
- Use the **recursive approach** for educational purposes or small values of `n`.
- Use the **memoized recursive approach** for larger values of `n` when recursion is preferred.

Let me know if you have further questions!
