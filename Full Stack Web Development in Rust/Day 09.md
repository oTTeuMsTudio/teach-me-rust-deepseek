You're building a **full-stack Rust web app** with Leptos, and you've already added:
- ✅ PostgreSQL for persistence
- ✅ JWT-based authentication
- ✅ Real-time updates with WebSockets
- ✅ AI-powered features (OpenAI)
- ✅ Mobile-first responsive UI with Tailwind
- ✅ Unit & integration tests

Now let’s add **structured logging** using `tracing` + `tracing-subscriber` so you can:
- 📊 Monitor your app in production
- 🔍 Debug issues easily
- 🧾 Log structured JSON data to files or observability platforms

---

# 📊 Add Structured Logging with `tracing` in Rust + Leptos

## ✅ What We’ll Do:

1. Set up `tracing_subscriber` for logging  
2. Add request logging middleware for Axum  
3. Add span tracing around key operations  
4. Log to stdout (for local dev) and optionally to file  
5. Add tracing context to server functions  

Let's go!

---

## 1. 🛠️ Add Dependencies

Add these to `my_todo_app_axum/Cargo.toml`:

```toml
[dependencies]
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }
tower-http = { version = "0.3", features = ["trace"] }
```

---

## 2. 🧰 Initialize Global Subscriber

Update `main.rs` to initialize the logger:

```rust
use tracing_subscriber::{layer::SubscriberExt, util::SubscriberInitExt};

#[tokio::main]
async fn main() {
    // Setup tracing
    tracing_subscriber::registry()
        .with(tracing_subscriber::EnvFilter::try_from_default_env()
            .unwrap_or_else(|_| "my_todo_app_axum=debug,tower_http=debug".into()))
        .with(tracing_subscriber::fmt::layer())
        .init();

    // Other setup...
}
```

Now logs will be printed to stdout.

---

## 3. 🌐 Add Request Tracing Middleware (Axum)

Use `TraceLayer` from `tower-http` to log HTTP requests:

In `main.rs`:

```rust
use tower_http::trace::TraceLayer;
use axum::Router;

#[tokio::main]
async fn main() {
    // ...

    // Build router with trace layer
    let app = Router::new()
        .leptos_routes(...)
        .fallback(...)
        .layer(TraceLayer::new_for_http());

    // ...
}
```

This gives you logs like:

```
DEBUG tower_http::trace::on_request: method=GET path="/api/todos"
DEBUG tower_http::trace::on_response: status=200 latency=15.2ms
```

---

## 4. 🧱 Add Tracing Context to Key Functions

Update your server functions to use spans:

```rust
use tracing::instrument;

#[server(GetTodos)]
#[instrument]
pub async fn get_todos() -> Result<Vec<Todo>, ServerFnError> {
    let pool = get_pool()?;
    sqlx::query_as!(Todo, r#"SELECT * FROM todos"#)
        .fetch_all(&pool)
        .await
        .map_err(|e| ServerFnError::new(e.to_string()))
}
```

Now each call to this function gets traced.

---

## 5. 🗃️ Optional: Log to File

Want to log to a file instead of just stdout?

Create a new file: `src/logging.rs`

```rust
use std::fs::File;
use tracing_subscriber::{fmt, prelude::*, EnvFilter};

pub fn init_file_logger() {
    let file = File::create("app.log").expect("Failed to create log file");

    tracing_subscriber::registry()
        .with(EnvFilter::from_default_env())
        .with(fmt::layer().with_writer(file))
        .init();
}
```

Then in `main.rs`:

```rust
#[tokio::main]
async fn main() {
    crate::logging::init_file_logger();

    // rest of the setup
}
```

---

## 6. 🧪 Bonus: Add OpenTelemetry Support (Optional)

For observability (like sending logs/metrics to Grafana, Datadog, etc):

```bash
cargo add opentelemetry tempfile reqwest-tungstenite
```

Then you can send traces to an OTLP collector:

```rust
use opentelemetry::sdk::export::trace::stdout;
use tracing_opentelemetry::layer as otel_layer;

opentelemetry::global::set_text_map_propagator(opentelemetry_jaeger::Propagator::new());
let tracer = stdout::new_pipeline().install_simple();
let otel_layer = otel_layer().with_tracer(tracer);

tracing_subscriber::registry()
    .with(otel_layer)
    .with(fmt::layer())
    .init();
```

Now your logs are exported with distributed tracing context.

---

## ✅ Summary

✅ Added structured logging with `tracing`  
✅ Enabled request tracing via Axum middleware  
✅ Used spans to trace server functions  
✅ Logged to stdout and optionally to file  
✅ Optionally enabled OpenTelemetry for observability  

---

## 🚀 Next:

1. 🔐 Securing headers with **Axum middleware**
2. 🧩 Creating a **dashboard or profile page**
3. 🧪 Writing **E2E tests** with Playwright or Selenium
4. 🧑‍💻 Adding user profiles and settings
