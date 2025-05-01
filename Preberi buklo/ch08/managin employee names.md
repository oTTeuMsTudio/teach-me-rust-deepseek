Here's a well-structured and fully functional implementation in Rust that satisfies the requirements of managing employee names in departments using a `HashMap` and `Vec`. The program provides a simple text-based interface for adding employees and listing them either per department or across the company, with all output sorted alphabetically.

---

### ✅ Features Implemented

- **Add Employees**: Accepts commands like `Add Sally to Engineering`.
- **List Department**: Outputs all employees in a specific department sorted alphabetically.
- **List All**: Outputs all employees grouped by department, with departments and names both sorted.
- **Case-sensitive department and employee names**.
- **Maintains sorted order internally** when adding names to avoid re-sorting on retrieval.

---

### 📦 Dependencies

This code uses only the standard library:

```rust
use std::collections::HashMap;
use std::io;
```

---

### 🧠 Implementation Details

- A `HashMap<String, Vec<String>>` is used to map department names to employee names.
- When adding an employee, the department vector is sorted immediately to maintain order.
- Listing operations retrieve and display the pre-sorted data.
- The input is parsed using `split_whitespace()` and basic pattern matching.

---

### 🧾 Full Code

```rust
use std::collections::HashMap;
use std::io;

fn main() {
    let mut company: HashMap<String, Vec<String>> = HashMap::new();

    loop {
        println!("Enter command (Add <name> to <department>, List <department>, or List all):");
        let mut input = String::new();
        io::stdin()
            .read_line(&mut input)
            .expect("Failed to read line");

        let tokens: Vec<&str> = input.trim().split_whitespace().collect();
        if tokens.is_empty() {
            continue;
        }

        match tokens[0] {
            "Add" => {
                if tokens.len() < 4 || tokens[2] != "to" {
                    println!("Invalid command format. Use 'Add <name> to <department>'");
                    continue;
                }
                let name = tokens[1].to_string();
                let department = tokens[3].to_string();

                let employees = company.entry(department)
                    .or_insert_with(Vec::new);
                employees.push(name);
                employees.sort(); // Keep the list sorted
            }
            "List" => {
                if tokens.len() < 2 {
                    println!("Invalid command format. Use 'List <department>' or 'List all'");
                    continue;
                }

                let target = tokens[1];

                if target == "all" {
                    let mut departments: Vec<String> = company.keys().cloned().collect();
                    departments.sort();

                    for dept in departments {
                        println!("Department: {}", dept);
                        if let Some(employees) = company.get(&dept) {
                            for name in employees {
                                println!(" - {}", name);
                            }
                        }
                    }
                } else {
                    if let Some(employees) = company.get(target) {
                        for name in employees {
                            println!("{}", name);
                        }
                    } else {
                        println!("Department '{}' not found.", target);
                    }
                }
            }
            _ => {
                println!("Unknown command. Available commands: Add, List");
            }
        }
    }
}
```

---

### 🧪 Example Usage

**Input:**
```
Add Sally to Engineering
Add Amir to Sales
Add Zoe to Engineering
List Engineering
List all
```

**Output:**
```
Department: Engineering
 - Sally
 - Zoe
Department: Sales
 - Amir
```

---

### 📌 Notes

- **Department and employee names are case-sensitive.**
- **No duplicate checks are performed.** If needed, you can add a check to avoid duplicates.
- **No persistence.** All data is lost on program exit.
- **Efficiency:** Sorting on each addition ensures fast retrieval but incurs a cost at insert time. For large datasets, consider sorting only at retrieval or using a `BTreeSet`.

---

This implementation provides a clean and effective solution to the problem, leveraging Rust's powerful ownership and pattern-matching features.
