# Enumerate all possible variants
---

### **What is this code doing?**
This Rust program defines two types of IP addresses (IPv4 and IPv6) and creates two specific examples of IP addresses: one for a "home" network and another for a "loopback" address.

---

### **Step-by-Step Explanation**

1. **Defining an `enum` for IP Address Types (`IpAddrKind`)**:
   ```rust
   enum IpAddrKind {
       V4,
       V6,
   }
   ```
   - An `enum` (short for "enumeration") is a way to define a type that can have a fixed set of values.
   - Here, `IpAddrKind` has two possible values: `V4` (for IPv4 addresses) and `V6` (for IPv6 addresses).
   - Think of it like a menu where you can choose between two options: "V4" or "V6".

2. **Defining a `struct` to Store IP Address Information (`IpAddr`)**:
   ```rust
   struct IpAddr {
       kind: IpAddrKind,
       address: String,
   }
   ```
   - A `struct` is like a blueprint for creating objects that hold related data.
   - The `IpAddr` struct has two fields:
     - `kind`: This tells us the type of IP address (either `V4` or `V6`), using the `IpAddrKind` enum.
     - `address`: This is the actual IP address as a `String`.

3. **Creating an Example IPv4 Address (`home`)**:
   ```rust
   let home = IpAddr {
       kind: IpAddrKind::V4,
       address: String::from("127.0.0.1"),
   };
   ```
   - Here, we create an instance of the `IpAddr` struct called `home`.
   - Its `kind` is set to `IpAddrKind::V4`, meaning it's an IPv4 address.
   - Its `address` is `"127.0.0.1"`, which is the standard "localhost" address for IPv4.

4. **Creating an Example IPv6 Address (`loopback`)**:
   ```rust
   let loopback = IpAddr {
       kind: IpAddrKind::V6,
       address: String::from("::1"),
   };
   ```
   - Similarly, we create another instance of the `IpAddr` struct called `loopback`.
   - Its `kind` is set to `IpAddrKind::V6`, meaning it's an IPv6 address.
   - Its `address` is `"::1"`, which is the standard "localhost" address for IPv6.

---

### **Why is this useful?**
- This code organizes IP address information in a structured way.
- By using an `enum` for the type of IP address (`V4` or `V6`), the program ensures that only valid types are used.
- The `struct` allows us to bundle both the type (`V4`/`V6`) and the address (`127.0.0.1` or `::1`) together into a single object.

---

### **Real-World Analogy**
Imagine you're organizing a list of phone numbers:
- You might have two types of phone numbers: **landline** and **mobile**.
- Each phone number also has an actual number (e.g., "123-456-7890").
- The `enum` (`IpAddrKind`) is like saying "this is a landline" or "this is a mobile".
- The `struct` (`IpAddr`) is like combining the type (landline/mobile) with the actual number into one entry.

---

### **Key Takeaways**
- The `enum` (`IpAddrKind`) defines the possible types of IP addresses.
- The `struct` (`IpAddr`) groups the type and the address into a single object.
- The program creates two examples:
  - `home`: An IPv4 address (`127.0.0.1`).
  - `loopback`: An IPv6 address (`::1`).

This approach makes the code clean, organized, and easy to extend if more IP address types are added in the future.

---

### **Final Answer**
This Rust program defines a system to represent IP addresses (both IPv4 and IPv6) using an `enum` and a `struct`. It creates two example IP addresses: `home` (IPv4: `127.0.0.1`) and `loopback` (IPv6: `::1`).
