# Building backend using Axum and Tokio

Let's dive into building a backend using **Axum** and **Tokio** in Rust. Axum is a modern, ergonomic web framework built on top of **Tower**, **Hyper**, and **Tokio**, making it highly performant and flexible for building web applications.

We'll cover the following steps:
1. Setting up the environment.
2. Creating a basic Axum server.
3. Adding routes and handlers.
4. Using asynchronous programming with Tokio.
5. Integrating middleware (optional).
6. Running the server.

---

### 1. Setting Up the Environment

First, ensure you have Rust installed. You can check this by running:

```bash
rustc --version
```

If not installed, follow the instructions at [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install).

Next, create a new Rust project:

```bash
cargo new axum_backend
cd axum_backend
```

Add the necessary dependencies to your `Cargo.toml` file:

```toml
[dependencies]
axum = "0.7"  # Latest version as of writing
tokio = { version = "1", features = ["full"] }
tower-http = "0.3"  # For middleware support
serde = { version = "1.0", features = ["derive"] }  # For JSON serialization/deserialization
```

---

### 2. Creating a Basic Axum Server

Open `src/main.rs` and start by setting up a simple Axum server:

```rust
use axum::{
    routing::get,
    Router,
};
use std::net::SocketAddr;

#[tokio::main]
async fn main() {
    // Define the router with a single route
    let app = Router::new()
        .route("/", get(root_handler));

    // Bind the server to a specific address
    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Server running on http://{}", addr);

    // Run the server
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

// A simple handler function
async fn root_handler() -> &'static str {
    "Hello, world!"
}
```

#### Explanation:
- `Router::new()` creates a new router.
- `.route("/", get(root_handler))` defines a route for the root URL (`/`) that responds with "Hello, world!".
- `tokio::main` ensures the application runs in an asynchronous context.
- `axum::Server::bind` binds the server to a specific IP and port.

Run the server:

```bash
cargo run
```

Navigate to `http://127.0.0.1:3000` in your browser or use `curl`:

```bash
curl http://127.0.0.1:3000
```

You should see:

```
Hello, world!
```

---

### 3. Adding Routes and Handlers

Let's add more routes and handlers. For example, we'll create a route that accepts query parameters and returns JSON.

Update `src/main.rs`:

```rust
use axum::{
    routing::{get, post},
    Json,
    extract::Query,
    response::IntoResponse,
};
use serde::Deserialize;

#[derive(Deserialize)]
struct GreetingParams {
    name: Option<String>,
}

async fn greet_handler(Query(params): Query<GreetingParams>) -> impl IntoResponse {
    let name = params.name.as_deref().unwrap_or("world");
    Json(serde_json::json!({"message": format!("Hello, {}!", name)}))
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(root_handler))
        .route("/greet", get(greet_handler));

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Server running on http://{}", addr);

    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

async fn root_handler() -> &'static str {
    "Hello, world!"
}
```

#### Explanation:
- We define a `GreetingParams` struct to deserialize query parameters.
- The `greet_handler` extracts the `name` parameter from the query string and returns a JSON response.
- The `/greet` route uses this handler.

Test it with `curl`:

```bash
curl "http://127.0.0.1:3000/greet?name=Alice"
```

Output:

```json
{"message":"Hello, Alice!"}
```

---

### 4. Using Asynchronous Programming with Tokio

Axum leverages Tokio for asynchronous operations. Let's simulate an async database call.

Add a new route that simulates fetching data from a database:

```rust
use tokio::time::sleep;
use std::time::Duration;

async fn fetch_data_from_db() -> String {
    sleep(Duration::from_secs(2)).await; // Simulate a delay
    "Data fetched from DB".to_string()
}

async fn db_handler() -> String {
    let data = fetch_data_from_db().await;
    format!("Database Response: {}", data)
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(root_handler))
        .route("/greet", get(greet_handler))
        .route("/db", get(db_handler));

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Server running on http://{}", addr);

    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

Now, hitting `http://127.0.0.1:3000/db` will take 2 seconds to respond due to the simulated delay.

---

### 5. Integrating Middleware (Optional)

Middlewares allow you to add functionality like logging, authentication, etc. Here's how to add a simple logging middleware:

```rust
use tower_http::trace::TraceLayer;

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(root_handler))
        .route("/greet", get(greet_handler))
        .route("/db", get(db_handler))
        .layer(TraceLayer::new_for_http()); // Add middleware

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Server running on http://{}", addr);

    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

This middleware logs request and response details.

---

### 6. Running the Server

Finally, run the server:

```bash
cargo run
```

You now have a fully functional Axum backend with Tokio handling asynchronous operations.

---

### Next Steps

- Learn about **state management** in Axum to share data across handlers.
- Explore **databases** like SQLx or MongoDB for persistent storage.
- Implement **authentication** and **authorization** using JWT or OAuth.
- Optimize performance with **connection pooling** and **caching**.

