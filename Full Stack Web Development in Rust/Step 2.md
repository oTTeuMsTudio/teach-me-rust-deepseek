# GitHub-ready template for a full-stack To-Do app in Rust using Leptos + Axum + SQLite + SQLx, complete with:

- Client-side interactivity (Leptos)
- Server-side API (Axum)
- Database persistence (SQLite + SQLx)
- Build system setup (Trunk, WASM)

---

## 📦 GitHub Repo Template: Full-Stack To-Do App in Rust

### 🔗 Repository Structure

```
leptos-todo-app/
├── Cargo.toml
├── src/
│   ├── main.rs        # Axum server + Leptos SSR/CSR
│   └── lib.rs         # Leptos components
├── db/
│   ├── schema.sql
│   └── todos.db       # SQLite DB
├── index.html
└── trunk.toml         # Trunk config
```

---

## 🧰 1. `Cargo.toml`

```toml
[package]
name = "leptos-todo-app"
version = "0.1.0"
edition = "2021"

[dependencies]
leptos = { version = "0.4", features = ["ssr", "csr"] }
axum = "0.6"
tokio = { version = "1", features = ["full"] }
sqlx = { version = "0.6", features = ["sqlite", "runtime-tokio-native-tls"] }
reqwasm = "0.5"
serde = "1.0"
serde_json = "1.0"
dotenvy = "0.15"
uuid = { version = "1.0", features = ["v4"] }

[lib]
crate-type = ["cdylib", "rlib"]
```

---

## 🖥️ 2. `src/lib.rs` — Leptos Frontend

```rust
use leptos::*;

#[component]
pub fn App() -> impl IntoView {
    view! {
        <h1>"Leptos Full-Stack To-Do App"</h1>
        <TodoApp />
    }
}

#[component]
fn TodoApp() -> impl IntoView {
    let fetch_todos = create_resource(|| (), |_| async move {
        reqwasm::http::Request::get("/api/todos")
            .send()
            .await
            .unwrap()
            .json::<Vec<String>>()
            .await
            .unwrap()
    });

    view! {
        <Suspense fallback=|| ()>
            {move || {
                fetch_todos.get().map(|todos| {
                    view! {
                        <ul class="todo-list">
                            {todos.into_iter().map(|t| view!{<li>{t}</li>}).collect::<Vec<_>>()}
                        </ul>
                    }
                })
            }}
        </Suspense>
    }
}
```

---

## 🌐 3. `src/main.rs` — Axum Backend + Leptos Setup

```rust
use axum::{
    Router,
    routing::{get, post},
};
use leptos::*;
use std::net::SocketAddr;
use sqlx::SqlitePool;
use std::sync::Arc;

mod app;
use app::*;

struct AppState {
    pool: SqlitePool,
}

#[tokio::main]
async fn main() {
    // Load environment variables
    dotenvy::dotenv().ok();

    // Initialize DB connection pool
    let pool = SqlitePool::connect("sqlite:db/todos.db").await.unwrap();
    sqlx::migrate!("db/migrations").run(&pool).await.unwrap();

    let state = Arc::new(AppState { pool });

    // Build the Leptos environment
    let conf = get_configuration(None).await.unwrap();
    let leptos_options = conf.leptos_options;

    let app = Router::new()
        .route("/", get(move || {
            let options = leptos_options.clone();
            move || generate_route(App, &options)
        }))
        .route("/api/todos", get(get_todos))
        .with_state(Arc::clone(&state));

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("🚀 Listening on http://{}", addr);
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

async fn get_todos(State(state): State<Arc<AppState>>) -> String {
    let todos = sqlx::query!("SELECT text FROM todos")
        .fetch_all(&state.pool)
        .await
        .unwrap();

    let texts: Vec<String> = todos.iter().map(|t| t.text.clone()).collect();
    serde_json::to_string(&texts).unwrap()
}
```

---

## 🗄️ 4. `db/schema.sql`

```sql
CREATE TABLE IF NOT EXISTS todos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    text TEXT NOT NULL
);
```

Create and initialize the database:

```bash
mkdir -p db
touch db/todos.db
sqlite3 db/todos.db < db/schema.sql
```

---

## 🧱 5. `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Leptos To-Do App</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

---

## ⚙️ 6. `trunk.toml`

```toml
[build]
target = "wasm32-unknown-unknown"
out-dir = "dist"
public-path = "/pkg"
```

---

## 🏃‍♂️ How to Run

```bash
# Step 1: Install dependencies
cargo install trunk wasm-pack

# Step 2: Create DB
mkdir -p db
touch db/todos.db
sqlite3 db/todos.db < db/schema.sql

# Step 3: Run dev server
trunk serve --port 3000
```

Then open [http://localhost:3000](http://localhost:3000)

---

## 🛠️ Next Steps You Can Add

- ✅ Add POST route to insert new todos
- ✅ Use `create_action()` to send form data from frontend
- ✅ Add delete functionality
- ✅ Add Tailwind CSS for styling
- ✅ Deploy to Cloudflare Workers + D1 or Supabase

