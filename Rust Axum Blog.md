Creating a Rust-based blog using the Axum web framework and deploying it on Leapcell.io is an excellent way to learn modern web development with Rust. Below, I will guide you step-by-step through the process of building a simple blog application and deploying it on Leapcell.io.

---

### **Step 1: Prerequisites**
Before we begin, ensure you have the following installed:
1. **Rust**: Install Rust from [rust-lang.org](https://www.rust-lang.org/tools/install).
2. **Cargo**: This comes bundled with Rust and is used for managing dependencies and building projects.
3. **Leapcell Account**: Sign up at [Leapcell.io](https://leapcell.io) to get access to their deployment platform.
4. **Git**: For version control and deployment.

---

### **Step 2: Initialize a New Rust Project**
Start by creating a new Rust project using Cargo:
```bash
cargo new rust-axum-blog
cd rust-axum-blog
```

Add the necessary dependencies to your `Cargo.toml` file:
```toml
[dependencies]
axum = "0.6"
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tower-http = "0.3"
```
- **Axum**: A lightweight web framework for Rust.
- **Tokio**: An asynchronous runtime for Rust.
- **Serde**: For JSON serialization and deserialization.
- **Tower-HTTP**: Middleware for handling HTTP requests.

Run `cargo build` to download and compile the dependencies.

---

### **Step 3: Define the Blog Data Structure**
Create a simple data structure for blog posts. Add the following code to `src/main.rs`:
```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct BlogPost {
    id: u32,
    title: String,
    content: String,
}
```

---

### **Step 4: Set Up Axum Routes**
Define routes for listing all blog posts, viewing a single post, and adding a new post. Update `src/main.rs` as follows:
```rust
use axum::{
    routing::{get, post},
    Router, Json, extract::State,
};
use std::sync::Arc;
use tokio::sync::Mutex;

type AppState = Arc<Mutex<Vec<BlogPost>>>;

#[tokio::main]
async fn main() {
    // Initialize shared state
    let app_state = Arc::new(Mutex::new(vec![
        BlogPost {
            id: 1,
            title: "First Post".to_string(),
            content: "This is the first blog post.".to_string(),
        },
        BlogPost {
            id: 2,
            title: "Second Post".to_string(),
            content: "This is the second blog post.".to_string(),
        },
    ]));

    // Define routes
    let app = Router::new()
        .route("/posts", get(list_posts).post(create_post))
        .route("/posts/:id", get(get_post))
        .with_state(app_state);

    // Start the server
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("Server running at http://localhost:3000");
    axum::serve(listener, app).await.unwrap();
}

// Route handlers
async fn list_posts(State(state): State<AppState>) -> Json<Vec<BlogPost>> {
    let posts = state.lock().await;
    Json(posts.clone())
}

async fn get_post(
    State(state): State<AppState>,
    axum::extract::Path(id): axum::extract::Path<u32>,
) -> Result<Json<BlogPost>, String> {
    let posts = state.lock().await;
    if let Some(post) = posts.iter().find(|p| p.id == id) {
        Ok(Json(post.clone()))
    } else {
        Err(format!("Post with id {} not found", id))
    }
}

async fn create_post(
    State(state): State<AppState>,
    Json(payload): Json<BlogPost>,
) -> Json<BlogPost> {
    let mut posts = state.lock().await;
    let new_id = posts.len() as u32 + 1;
    let new_post = BlogPost {
        id: new_id,
        title: payload.title,
        content: payload.content,
    };
    posts.push(new_post.clone());
    Json(new_post)
}
```

---

### **Step 5: Test Locally**
Run the server locally:
```bash
cargo run
```
Visit the following endpoints in your browser or using `curl`:
1. **List all posts**: `GET http://localhost:3000/posts`
2. **Get a specific post**: `GET http://localhost:3000/posts/1`
3. **Create a new post**: `POST http://localhost:3000/posts` with a JSON body like:
   ```json
   {
       "title": "New Post",
       "content": "This is a new blog post."
   }
   ```

---

### **Step 6: Prepare for Deployment**
1. **Build the Project**:
   Compile the project for release:
   ```bash
   cargo build --release
   ```
   The binary will be located at `target/release/rust-axum-blog`.

2. **Create a Dockerfile**:
   Create a `Dockerfile` in the root of your project:
   ```dockerfile
   FROM rust:1.72 AS builder
   WORKDIR /app
   COPY . .
   RUN cargo build --release

   FROM debian:bullseye-slim
   WORKDIR /app
   COPY --from=builder /app/target/release/rust-axum-blog .
   CMD ["./rust-axum-blog"]
   ```

3. **Build the Docker Image**:
   Build the Docker image:
   ```bash
   docker build -t rust-axum-blog .
   ```

---

### **Step 7: Deploy to Leapcell.io**
1. **Push the Docker Image**:
   Push your Docker image to a container registry (e.g., Docker Hub or GitHub Packages).

2. **Deploy on Leapcell**:
   - Log in to your Leapcell account.
   - Create a new project and select "Containerized Application".
   - Provide the Docker image URL and configure any environment variables if needed.
   - Deploy the application.

---

### **Step 8: Verify Deployment**
Once deployed, Leapcell will provide you with a URL where your blog is hosted. Visit the URL to verify that everything works as expected.

---

### **Conclusion**
You now have a fully functional Rust-based blog using Axum, deployed on Leapcell.io. This setup can be extended with additional features such as:
- Persistent storage (e.g., SQLite, PostgreSQL).
- Frontend integration (e.g., React, Vue.js).
- Authentication and authorization.
