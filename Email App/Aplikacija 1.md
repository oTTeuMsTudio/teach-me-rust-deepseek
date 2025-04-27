# A Rust application that discovers and verifies professional email addresses 
based on contact names and company websites, we need to break the problem into several steps:

1. **Input Parsing**: Accept contact names and company domains as input.
2. **Email Pattern Generation**: Generate possible email patterns using common conventions (e.g., `first.last@company.com`, `f.last@company.com`).
3. **Domain Validation**: Verify if the company domain has valid MX records (Mail Exchange records) to ensure it can receive emails.
4. **Email Verification**: Use SMTP (Simple Mail Transfer Protocol) to check if the generated email addresses are valid without actually sending an email.

Below is a step-by-step guide to building this tool in Rust:

---

### Step 1: Set Up the Project
Create a new Rust project using Cargo:
```bash
cargo new email_discovery_tool
cd email_discovery_tool
```

Add necessary dependencies to your `Cargo.toml` file:
```toml
[dependencies]
reqwest = "0.11" # For HTTP requests
tokio = { version = "1", features = ["full"] } # For async runtime
dns-lookup = "0.7" # For DNS lookups
serde = { version = "1.0", features = ["derive"] } # For JSON parsing (if needed)
regex = "1.8" # For pattern matching
```

---

### Step 2: Input Parsing
Write a function to accept contact names and company domains. For simplicity, assume the input is provided as command-line arguments or via a JSON file.

Example input:
```json
{
  "contacts": [
    {"name": "John Doe", "company_domain": "example.com"},
    {"name": "Jane Smith", "company_domain": "test.com"}
  ]
}
```

Parse the input using Serde:
```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct Contact {
    name: String,
    company_domain: String,
}

#[derive(Deserialize)]
struct Input {
    contacts: Vec<Contact>,
}

fn parse_input(json_data: &str) -> Result<Input, serde_json::Error> {
    serde_json::from_str(json_data)
}
```

---

### Step 3: Email Pattern Generation
Generate common email patterns based on the contact's name and company domain:
```rust
fn generate_email_patterns(name: &str, domain: &str) -> Vec<String> {
    let parts: Vec<&str> = name.split_whitespace().collect();
    let first = parts[0].to_lowercase();
    let last = parts[1].to_lowercase();

    vec![
        format!("{}@{}", first, domain),
        format!("{}.{}@{}", first, last, domain),
        format!("{}{}@{}", first.chars().next().unwrap(), last, domain),
        format!("{}@{}", last, domain),
        format!("{}{}@{}", last, first.chars().next().unwrap(), domain),
    ]
}
```

---

### Step 4: Domain Validation
Check if the company domain has valid MX records:
```rust
use dns_lookup::lookup_mx;

fn validate_domain(domain: &str) -> bool {
    match lookup_mx(domain) {
        Ok(mx_records) => !mx_records.is_empty(),
        Err(_) => false,
    }
}
```

---

### Step 5: Email Verification Using SMTP
Verify email addresses by connecting to the SMTP server of the domain and checking if the address exists. This requires interacting with the SMTP protocol.

Here’s a simplified implementation:
```rust
use std::net::TcpStream;
use std::io::{BufRead, BufReader, Write};

fn verify_email(email: &str, domain: &str) -> bool {
    // Connect to the SMTP server
    let server = format!("smtp.{}", domain);
    if let Ok(stream) = TcpStream::connect(format!("{}:25", server)) {
        let mut reader = BufReader::new(&stream);
        let mut writer = stream;

        // Read the initial server response
        let mut line = String::new();
        if reader.read_line(&mut line).is_err() || !line.starts_with("220") {
            return false;
        }

        // Send HELO command
        writeln!(writer, "HELO example.com").unwrap();
        line.clear();
        if reader.read_line(&mut line).is_err() || !line.starts_with("250") {
            return false;
        }

        // Send MAIL FROM command
        writeln!(writer, "MAIL FROM:<test@example.com>").unwrap();
        line.clear();
        if reader.read_line(&mut line).is_err() || !line.starts_with("250") {
            return false;
        }

        // Send RCPT TO command to check the email
        writeln!(writer, "RCPT TO:<{}>", email).unwrap();
        line.clear();
        if reader.read_line(&mut line).is_err() || !line.starts_with("250") {
            return false;
        }

        true
    } else {
        false
    }
}
```

---

### Step 6: Putting It All Together
Combine all the components into a main function:
```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let json_data = r#"
    {
        "contacts": [
            {"name": "John Doe", "company_domain": "example.com"},
            {"name": "Jane Smith", "company_domain": "test.com"}
        ]
    }
    "#;

    let input = parse_input(json_data)?;

    for contact in input.contacts {
        let domain = &contact.company_domain;

        if !validate_domain(domain) {
            println!("Invalid domain: {}", domain);
            continue;
        }

        let patterns = generate_email_patterns(&contact.name, domain);

        for email in patterns {
            if verify_email(&email, domain) {
                println!("Valid email found: {}", email);
            } else {
                println!("Invalid email: {}", email);
            }
        }
    }

    Ok(())
}
```

---

### Notes and Considerations
1. **SMTP Verification Limitations**: Some servers may not allow RCPT TO checks, or they may block repeated attempts. Handle these cases gracefully.
2. **Rate Limiting**: Add delays between requests to avoid being blocked by servers.
3. **Error Handling**: Improve error handling for network issues and invalid inputs.
4. **Ethical Use**: Ensure compliance with privacy laws and terms of service when using this tool.

This implementation provides a foundation for discovering and verifying professional email addresses. You can extend it further by adding logging, caching, or integrating with APIs for additional data sources.
