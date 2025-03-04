# Write a web template in Rust

We'll cover the basics of setting up an Actix-web server, handling CORS, working with `serde` for serialization/deserialization, making HTTP requests with `reqwest`, and managing shared state with `Mutex`.

---

### **1. Setting Up the Project**

First, create a new Rust project:

```bash
cargo new web_template --bin
cd web_template
```

Add the necessary dependencies to your `Cargo.toml` file:

```toml
[dependencies]
actix-web = "4" # Web framework
actix-cors = "0.6" # CORS middleware
serde = { version = "1.0", features = ["derive"] } # Serialization/Deserialization
reqwest = { version = "0.11", features = ["json"] } # HTTP client
tokio = { version = "1", features = ["full"] } # Async runtime
async-trait = "0.1" # For async traits
```

---

### **2. Importing Libraries**

Here’s how you can import the required libraries at the top of your `main.rs` file:

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use actix_cors::Cors; // CORS middleware
use serde::{Serialize, Deserialize}; // Serialization/Deserialization
use reqwest::Client; // HTTP client
use async_trait::async_trait; // Async trait support
use std::sync::Mutex; // Thread-safe shared state
use std::collections::HashMap; // Key-value storage
use std::fs::File; // File operations
use std::io::Write; // Writing to files
```

---

### **3. Defining Shared State**

We’ll use a `HashMap` wrapped in a `Mutex` to store shared state across requests. This is useful for scenarios like caching or maintaining session data.

```rust
struct AppState {
    data: Mutex<HashMap<String, String>>, // Thread-safe shared state
}

// Initialize the app state
fn init_state() -> AppState {
    AppState {
        data: Mutex::new(HashMap::new()),
    }
}
```

---

### **4. Creating Routes**

Define some routes for your web application. For example, let’s create a simple REST API with endpoints for getting and setting key-value pairs.

#### **Route Handlers**

```rust
// Handler for GET /data/{key}
async fn get_data(
    path: web::Path<String>,
    data: web::Data<AppState>,
) -> impl Responder {
    let key = path.into_inner();
    if let Some(value) = data.data.lock().unwrap().get(&key) {
        HttpResponse::Ok().json(value)
    } else {
        HttpResponse::NotFound().body("Key not found")
    }
}

// Handler for POST /data/{key}
async fn set_data(
    path: web::Path<String>,
    body: web::Json<String>,
    data: web::Data<AppState>,
) -> impl Responder {
    let key = path.into_inner();
    let value = body.into_inner();

    data.data.lock().unwrap().insert(key, value);

    HttpResponse::Created().body("Data stored successfully")
}
```

---

### **5. Making External HTTP Requests**

Let’s add a route that makes an external HTTP request using `reqwest`. For example, fetching data from a public API.

```rust
// Handler for GET /external
async fn fetch_external_data(client: web::Data<Client>) -> impl Responder {
    let url = "https://jsonplaceholder.typicode.com/posts/1";
    match client.get(url).send().await {
        Ok(response) => {
            if let Ok(body) = response.json::<serde_json::Value>().await {
                HttpResponse::Ok().json(body)
            } else {
                HttpResponse::InternalServerError().body("Failed to parse response")
            }
        }
        Err(_) => HttpResponse::InternalServerError().body("Failed to fetch data"),
    }
}
```

---

### **6. Configuring the Server**

Now, let’s configure the Actix-web server with CORS support and register our routes.

```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // Create a shared HTTP client
    let client = Client::new();

    // Initialize app state
    let app_state = web::Data::new(init_state());

    // Start the server
    HttpServer::new(move || {
        let cors = Cors::default()
            .allow_any_origin() // Allow all origins (for development purposes)
            .allowed_methods(vec!["GET", "POST"])
            .allowed_headers(vec![actix_web::http::header::CONTENT_TYPE]);

        App::new()
            .app_data(app_state.clone()) // Share app state
            .app_data(web::Data::new(client.clone())) // Share HTTP client
            .wrap(cors) // Enable CORS
            .route("/data/{key}", web::get().to(get_data))
            .route("/data/{key}", web::post().to(set_data))
            .route("/external", web::get().to(fetch_external_data))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

---

### **7. Writing Data to Files**

If you want to persist data to a file, you can use `std::fs::File` and `std::io::Write`. Here’s an example of writing data to a file:

```rust
fn write_to_file(data: &str) -> std::io::Result<()> {
    let mut file = File::create("output.txt")?;
    file.write_all(data.as_bytes())?;
    Ok(())
}
```

You can call this function from one of your route handlers if needed.

---

### **8. Running the Application**

To run your application, execute the following command:

```bash
cargo run
```

Your server will start on `http://127.0.0.1:8080`. You can test the endpoints using tools like `curl` or Postman.

---

### **9. Example Usage**

- **Set Data**: Send a POST request to `/data/<key>` with a JSON body containing the value.
  ```bash
  curl -X POST http://127.0.0.1:8080/data/my_key -H "Content-Type: application/json" -d '"my_value"'
  ```

- **Get Data**: Send a GET request to `/data/<key>`.
  ```bash
  curl http://127.0.0.1:8080/data/my_key
  ```

- **Fetch External Data**: Send a GET request to `/external`.
  ```bash
  curl http://127.0.0.1:8080/external
  ```

---

This template provides a solid foundation for building web applications in Rust using Actix-web. You can expand it further by adding authentication, database integration, or more complex business logic as needed.
