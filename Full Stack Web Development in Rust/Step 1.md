# Building a full-stack web application in Rust using Leptos, the modern, reactive framework for building front-end and full-stack apps with Rust

Go step-by-step:

---

## 🧠 What is Leptos?

**Leptos** is a **Rust web framework** that allows you to build both frontend (client-side) and backend (server-side) applications using the same codebase. It uses **WebAssembly (Wasm)** on the client side and can render server-side via **SSR (Server Side Rendering)** or **SSG (Static Site Generation)**. You write everything in **Rust**.

### Key Features:
- Reactive programming model
- Works with WebAssembly and SSR
- No need for JavaScript
- Full-stack capabilities

---

## 🛠️ Prerequisites

Make sure you have:

1. **Rust**: Installed via [rustup.rs](https://rustup.rs/)
2. **wasm-pack**: Install with `cargo install wasm-pack`
3. **Trunk**: Install with `cargo install trunk`

Optional:
- A database like PostgreSQL or SQLite
- Diesel or SQLx for ORM
- Axum or Actix-web for backend APIs

---

# 🚀 Step-by-Step: Full Stack App in Rust with Leptos

Create a simple **To-Do List app** that works fully in the browser but also persists data to a backend database.

---

## 📁 1. Create a New Project

Use `cargo generate` to scaffold a new Leptos project:

```bash
cargo install cargo-generate
cargo generate --git https://github.com/leptos-rs/start-parcel.git --name my-todo-app
cd my-todo-app
```

This will give basic structure:

```
my-todo-app/
├── Cargo.toml
├── src/
│   ├── main.rs        // entry point
│   └── lib.rs         // app components
└── index.html
```

---

## 🖥️ 2. Understanding the Structure

Leptos apps are structured around **components** and **reactivity**.

### Example Component (`src/lib.rs`)

```rust
use leptos::*;

#[component]
fn App() -> impl IntoView {
    let (count, set_count) = create_signal(0);

    view! {
        <button on:click=move |_| set_count.update(|n| *n += 1)>
            "Click me: "
            {count}
        </button>
    }
}

pub fn main() {
    leptos::mount_to_body(App);
}
```

You run it with:

```bash
trunk serve
```

Then open http://localhost:8080

---

## 🌐 3. Add Server-Side Functionality (Full-Stack)

Leptos integrates with **Axum** (or Actix-web) for backend functionality.

### a. Add Dependencies

In `Cargo.toml`, add:

```toml
[dependencies]
leptos = { version = "0.4", features = ["csr", "ssr"] }
axum = "0.6"
tokio = { version = "1", features = ["full"] }
serde = "1.0"
serde_json = "1.0"
```

### b. Setup Axum + Leptos

Update `main.rs`:

```rust
use axum::{
    Router,
    routing::get,
};
use leptos::*;
use std::net::SocketAddr;

mod app;
use app::*;

#[tokio::main]
async fn main() {
    // Build the Leptos environment
    let conf = get_configuration(None).await.unwrap();
    let leptos_options = conf.leptos_options;

    let app = Router::new()
        .route("/", get(move || {
            let options = leptos_options.clone();
            move || generate_route(App, &options)
        }));

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Listening on http://{}", addr);
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

### c. Define Routes in `lib.rs`

```rust
use leptos::*;

#[component]
fn App() -> impl IntoView {
    view! {
        <h1>"Welcome to Leptos!"</h1>
        <TodoApp />
    }
}

#[component]
fn TodoApp() -> impl IntoView {
    let (todos, set_todos) = create_signal(vec!["Buy milk".to_string(), "Walk dog".to_string()]);
    
    view! {
        <ul>
            {move || todos().into_iter().map(|todo| {
                view! { <li>{todo}</li> }
            }).collect::<Vec<_>>()}
        </ul>
    }
}
```

Now run:

```bash
trunk serve --port 3000
```

Go to http://localhost:3000

---

## 🗄️ 4. Connect to a Database (Optional)

Use **SQLite** and **SQLx** to persist todo items.

### a. Add SQLx

```toml
sqlx = { version = "0.6", features = ["sqlite", "runtime-tokio-native-tls"] }
dotenvy = "0.15"
```

### b. Set Up DB Schema

Create `db/schema.sql`:

```sql
CREATE TABLE IF NOT EXISTS todos (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    text TEXT NOT NULL
);
```

Initialize DB:

```bash
mkdir -p db
touch db/todos.db
sqlite3 db/todos.db < db/schema.sql
```

### c. Query from Axum Route

Add this to `main.rs`:

```rust
use sqlx::SqlitePool;
use axum::extract::State;
use std::sync::Arc;

struct AppState {
    pool: SqlitePool,
}

#[tokio::main]
async fn main() {
    // Initialize DB pool
    let pool = SqlitePool::connect("sqlite:db/todos.db").await.unwrap();

    // Migrate DB
    sqlx::migrate!("db/migrations").run(&pool).await.unwrap();

    let state = Arc::new(Arc::new(AppState { pool }));

    let app = Router::new()
        .route("/api/todos", get(get_todos))
        .with_state(state);

    // ...rest of main
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

### d. Fetch Data in Frontend

In `lib.rs`:

```rust
#[component]
fn TodoApp() -> impl IntoView {
    let fetch_todos = create_resource(|| (), |_| async move {
        let resp = reqwasm::http::Request::get("/api/todos")
            .send()
            .await
            .unwrap();

        resp.json::<Vec<String>>().await.unwrap()
    });

    view! {
        <Suspense fallback=|| ()>
            {move || {
                fetch_todos.get().map(|todos| {
                    view! {
                        <ul>
                            {todos.into_iter().map(|t| view!{<li>{t}</li>}).collect::<Vec<_>>()}
                        </ul>
                    }
                })
            }}
        </Suspense>
    }
}
```

Install `reqwasm`:

```toml
reqwasm = "0.5"
```

---

## 🧪 5. Testing and Debugging

Leptos supports logging with `web-sys`.

```rust
use web_sys::console;

console::log_1(&"Hello from Rust!".into());
```

For unit testing:

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn test_add() {
        assert_eq!(2 + 2, 4);
    }
}
```

Run tests:

```bash
cargo test
```

---

## 📦 6. Deployment

Leptos builds into WASM and HTML/CSS/JS bundles. Use Trunk to build:

```bash
trunk build --release
```

Can deploy the contents of the `dist/` folder to any static host (e.g., Vercel, Netlify, Cloudflare Pages).

If deploying the backend (Axum), make sure host supports running Rust binaries (like Docker or a Linux VM).

---

## 🧩 Bonus: Add Styling

You can use Tailwind CSS or plain CSS.

To add Tailwind:

1. Install Tailwind CLI
2. Configure `tailwind.config.js`
3. Add CDN link in `index.html`

---

## 🧠 Summary: Full Stack Architecture with Leptos

| Layer | Tech |
|------|------|
| Language | Rust |
| Frontend | Leptos + WebAssembly |
| Backend | Axum |
| DB | SQLite / PostgreSQL |
| ORM | SQLx / Diesel |
| HTTP Client | reqwasm |
| Build Tool | Trunk |
| Styling | TailwindCSS (optional) |

---

## 📘 Resources

- 🔗 Official Docs: [https://leptos.dev](https://leptos.dev)
- 📘 Book (Work-in-progress): [https://book.leptos.dev](https://book.leptos.dev)
- 💬 Discord: [https://discord.gg/6U4ZtjwQ6q](https://discord.gg/6U4ZtjwQ6q)
- 🎓 Examples: [https://github.com/leptos-rs/leptos/tree/main/examples](https://github.com/leptos-rs/leptos/tree/main/examples)

---

## ✅ Next Steps

1. Implement adding/deleting todos (frontend ↔ API ↔ DB)
2. Add authentication (JWT or OAuth)
3. Deploy to production (Cloudflare Workers + D1, Supabase, etc.)
4. Explore SSR/SSG with Leptos' meta tags, SEO support
