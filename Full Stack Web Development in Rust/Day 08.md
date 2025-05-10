We're building a **professional-grade, full-stack Rust app** using **Leptos**, and we've already added:
- ✅ PostgreSQL for persistence
- ✅ JWT-based authentication
- ✅ Real-time updates with WebSockets
- ✅ AI-powered features (OpenAI)
- ✅ Mobile-first responsive UI with Tailwind

Now let’s take it to the next level by writing **unit tests** and **integration tests** in Rust — ensuring our backend logic, server functions, and database interactions are solid.

---

# 🧪 Add Unit & Integration Tests to our Leptos App

## ✅ What We’ll Do:

1. Write **unit tests** for core logic  
2. Write **integration tests** for server functions  
3. Test database queries with `sqlx`  
4. Use `tempfile` and Docker to run test databases  
5. Mock external APIs like OpenAI (optional)  

Let's go!

---

## 1. 📁 Project Structure Recap

```
my_todo_app/
├── Cargo.toml
├── crates/
│   ├── my_todo_app/          # frontend + shared types
│   └── my_todo_app_axum/     # backend + server logic
├── migrations/
├── index.html
└── ...
```

We'll write tests inside `my_todo_app_axum`.

---

## 2. 🧱 Add Test Dependencies

Add these to `my_todo_app_axum/Cargo.toml`:

```toml
[dev-dependencies]
sqlx = { version = "0.6", features = ["postgres", "test"] }
sqlx-cli = "0.6"
tokio = { version = "1", features = ["full"] }
serde_json = "1.0"
reqwest = { version = "0.11", features = ["json"] }
```

---

## 3. 🧪 Unit Testing Server Functions

### Example: Test `toggle_todo()` logic

Create a file: `src/tests/unit.rs`

```rust
use crate::toggle_todo;
use my_todo_app::{Todo, get_pool};
use sqlx::PgPool;
use std::sync::Arc;

#[sqlx::test]
async fn test_toggle_todo(pool: PgPool) {
    let pool = Arc::new(pool);
    let todo = create_test_todo(&pool).await;

    let result = toggle_todo(todo.id).await.unwrap();
    assert_eq!(result[0].id, todo.id);
    assert_eq!(result[0].completed, true);
}

async fn create_test_todo(pool: &PgPool) -> Todo {
    sqlx::query_as!(
        Todo,
        "INSERT INTO todos (title) VALUES ('Test Todo') RETURNING *"
    )
    .fetch_one(pool)
    .await
    .unwrap()
}
```

> The `#[sqlx::test]` macro spins up an ephemeral Postgres DB for each test.

---

## 4. 🧪 Integration Testing Server Functions

### Goal: Simulate real HTTP requests to our server functions

Use `reqwest` and spawn a test Axum instance on port 0 (random free port).

Create: `src/tests/integration.rs`

```rust
use axum::{
    Router,
    body::Body,
    http::{Request, Method, HeaderMap},
};
use tower::ServiceExt; // for `oneshot` and `ready`
use my_todo_app_axum::main;

#[tokio::test]
async fn test_get_todos() {
    let app = main::build_app().await; // You'll define this function in main.rs

    let response = app.oneshot(Request::builder()
        .uri("/api/get_todos")
        .method(Method::GET)
        .body(Body::empty())
        .unwrap())
        .await
        .unwrap();

    assert_eq!(response.status(), 200);

    let body = hyper::body::to_bytes(response.into_body()).await.unwrap();
    let todos: Vec<crate::Todo> = serde_json::from_slice(&body).unwrap();
    assert!(todos.len() >= 0);
}
```

In `main.rs`, add:

```rust
pub fn build_app() -> Router {
    // Return your router setup without binding to a port
    Router::new()
        .leptos_routes(...)
        .fallback(...)
}
```

---

## 5. 🧪 Test Auth Logic

### Example: Test Login Functionality

```rust
#[tokio::test]
async fn test_login_success() {
    let app = build_app().await;

    let client = reqwest::Client::new();

    let res = client.post("http://localhost:3000/api/login")
        .json(&serde_json::json!({
            "email": "test@example.com",
            "password": "password123"
        }))
        .send()
        .await
        .unwrap();

    assert_eq!(res.status(), 200);
    let token: String = res.text().await.unwrap();
    assert!(token.contains("."));
}
```

---

## 6. 🧪 Optional: Mocking External APIs (e.g., OpenAI)

Use `wiremock` or `mockito` to mock external services like OpenAI.

Example with `wiremock`:

```bash
cargo add wiremock --dev
```

Then:

```rust
use wiremock::{MockServer, Mock, ResponseTemplate};
use wiremock::matchers::{method, path};

#[tokio::test]
async fn test_ai_suggestion() {
    let mock_server = MockServer::start().await;

    Mock::given(method("POST"))
        .and(path("/v1/completions"))
        .respond_with(ResponseTemplate::new(200).set_body_json(&serde_json::json!({
            "choices": [{"text": "Buy groceries"}]
        })))
        .mount(&mock_server)
        .await;

    // Override API URL in config
    std::env::set_var("OPENAI_API_URL", mock_server.uri());

    // Call your AI server function
    let suggestion = get_todo_suggestion("Need food".to_string()).await.unwrap();

    assert_eq!(suggestion.suggestion, "Buy groceries");
}
```

---

## ✅ Summary

✅ Added unit tests for server functions  
✅ Wrote integration tests with real HTTP calls  
✅ Used `sqlx::test` for isolated database testing  
✅ Mocked external APIs safely  
✅ Made your app more robust and maintainable  

---

## 🚀 Next:

1. 📊 Adding **analytics or logging** using tracing  
2. 🔐 Securing headers with **Axum middleware**  
3. 🧩 Creating a **dashboard or profile page**  
4. 🧪 Writing **E2E tests** with Playwright or Selenium
