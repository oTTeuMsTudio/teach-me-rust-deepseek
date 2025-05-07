Let's walk through **learning Full Stack Web Development using Leptos** — a modern, reactive Rust framework for building web applications. We'll start from the basics and work our way up to a full-stack app with backend and frontend in Rust.

---

## 🚀 What is Leptos?

Leptos is a **Rust-based web framework** that allows you to write **reactive UIs** directly in Rust. It uses a virtual DOM diffing algorithm and compiles your Rust code into WASM (WebAssembly), allowing you to build fast, interactive frontends. Leptos also supports server-side rendering (SSR), static site generation (SSG), and even full-stack apps using Axum or other backends.

### 🔧 Key Features:
- Built with **Rust**
- Uses **WebAssembly (WASM)** on the client
- Supports **SSR**, **CSR**, and hybrid approaches
- Reactive programming model similar to SolidJS
- Integrates well with **Axum** for backend

---

## 📦 Prerequisites

Before we begin:

- Basic knowledge of **Rust**
- Install **Rust**: https://rustup.rs/
- Install `wasm-pack`:  
  ```bash
  cargo install wasm-pack
  ```
- Install `cargo-leptos` CLI tool:  
  ```bash
  cargo install --locked cargo-leptos
  ```

---

## 🛠️ Step-by-Step Guide to Full Stack Web Dev with Leptos

---

### ✅ Step 1: Set Up Your Project

Let’s create a new full-stack project using Leptos and Axum.

```bash
cargo leptos new my_app --git
```

Choose the **Fullstack App (with Axum)** template.

This will scaffold a project with:
- Frontend in `src/lib.rs`
- Backend in `src/main.rs`
- Shared types in `src/shared.rs`
- A `Cargo.toml` that builds both sides

Then run:

```bash
cd my_app
npm install  # Installs dev dependencies
npm run dev
```

Your app should be running at http://localhost:3000

---

### 🧱 Step 2: Understand the Project Structure

```
my_app/
├── Cargo.toml
├── src/
│   ├── lib.rs        <-- Leptos frontend components
│   ├── main.rs        <-- Axum backend server
│   └── shared.rs      <-- Shared types between front/back ends
├── npm/
│   └── package.json
└── ...
```

---

### 💡 Step 3: Build a Simple Counter (Frontend)

Open `src/lib.rs`. This is where your Leptos components live.

Replace the content with this simple counter:

```rust
use leptos::*;

#[component]
pub fn App(cx: Scope) -> impl IntoView {
    let (count, set_count) = create_signal(cx, 0);

    view! { cx,
        <main>
            <h1>"Counter"</h1>
            <p>"Value: " {move || count.get()}</p>
            <button on:click=move |_| set_count.update(|n| *n += 1)>"Increment"</button>
        </main>
    }
}
```

That’s it! You just created a reactive component in Rust 🎉

---

### 🌐 Step 4: Add Server-Side Rendering (SSR)

Let’s add a simple API endpoint to get the current count from the server.

#### 1. Define a shared struct

In `src/shared.rs`:

```rust
#[derive(Clone, Debug, serde::Serialize, serde::Deserialize)]
pub struct AppState {
    pub initial_count: i32,
}
```

#### 2. Modify the backend (`src/main.rs`):

```rust
use axum::{Router, routing::get};
use leptos::*;
use my_app::*;
use std::sync::Arc;
use tokio::net::TcpListener;

#[tokio::main]
async fn main() {
    // Setup logging
    _ = console_log::init_with_level(log::LevelFilter::Debug);
    
    // Initial state
    let state = Arc::new(AppState { initial_count: 42 });

    // Build routes
    let routes = Router::new()
        .route("/api/count", get(|| async { format!("{}", state.initial_count) }))
        .leptos_routes::<SiteMap>(state.clone(), |cx| view! { cx, <App/> })
        .fallback(leptos_axum::file_and_error_handler(state.clone(), |cx| view! { cx, <App/> }));

    // Start server
    let listener = TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, routes.into_make_service())
        .await
        .unwrap();
}
```

#### 3. Fetch data from the frontend

Update `src/lib.rs`:

```rust
use leptos::*;
use gloo_net::http::Request;

#[component]
pub fn App(cx: Scope) -> impl IntoView {
    let count = create_resource(cx, || (), |_| async move {
        let text = Request::get("/api/count").send().await.unwrap().text().await.unwrap();
        text.parse::<i32>().unwrap_or(0)
    });

    view! { cx,
        <main>
            <h1>"Server-Side Count"</h1>
            <Suspense fallback=move || "Loading...">
                <p>"Initial count from server: " {move || count.read(cx)}</p>
            </Suspense>
        </main>
    }
}
```

Now you're fetching data from the Rust backend!

---

### 🗄️ Step 5: Connect to a Database (Backend)

Let’s connect to PostgreSQL using `sqlx`.

#### 1. Add dependencies to `Cargo.toml` (backend):

```toml
[dependencies]
sqlx = { version = "0.6", features = ["postgres", "runtime-tokio-native-tls"] }
dotenvy = "0.15"
```

#### 2. Create `.env` file:

```env
DATABASE_URL=postgres://user:password@localhost/mydb
```

#### 3. Query DB in your endpoint:

```rust
use sqlx::PgPool;

// In main.rs:
let pool = PgPool::connect("postgres://user:password@localhost/mydb").await.unwrap();

let routes = Router::new()
    .route("/api/users", get(move || {
        let pool = pool.clone();
        async move {
            let rows = sqlx::query!("SELECT id, name FROM users")
                .fetch_all(&pool)
                .await
                .unwrap();
            
            format!("{:#?}", rows)
        }
    }));
```

You now have a working full-stack Rust app with database access!

---

### 🧪 Step 6: Add Authentication (Optional)

You can use middleware like `tower-sessions` or integrate JWT authentication via `jsonwebtoken`.

Example with sessions:

```rust
use tower_sessions::{SessionManagerLayer, Session, MemoryStore};

let store = MemoryStore::default();
let session_layer = SessionManagerLayer::new(store);

let routes = Router::new()
    .layer(session_layer)
    .route("/login", post(login_handler));
```

---

### 📦 Step 7: Deploy Your App

Leptos works great with platforms that support WASM and Rust:

- **Vercel + Cloudflare Workers** (for SSR/WASM edge hosting)
- **Netlify / Vercel** (for static files)
- **Docker container** (for self-hosting)

Use `npm run build` to generate production assets.

---

## 🧠 Summary of Concepts

| Feature         | Tool/Technique              |
|----------------|----------------------------|
| Frontend       | Leptos + WASM               |
| Backend        | Axum                        |
| State Management | Signals, Resources          |
| Routing        | Leptos Router               |
| DB Access      | SQLx                        |
| Auth           | Tower Sessions / JWT        |
| Deployment     | WASM bundler + Docker/Nginx |

---

## 📚 Resources

- Official Docs: [https://leptos.dev](https://leptos.dev)
- GitHub: [https://github.com/gbj/leptos](https://github.com/gbj/leptos)
- Book: [https://book.leptos.dev](https://book.leptos.dev)
- Discord: [Leptos Discord](https://discord.gg/6U4ZtjwQ6q)

---

## 🏁 Final Thoughts

Leptos is powerful because it lets you write **entire web apps in Rust** — no need for JavaScript unless you're integrating external libraries.

As Rust adoption grows in web dev, frameworks like Leptos are pushing the boundaries of what's possible in terms of performance, safety, and developer experience.

Would you like me to help you build a specific full-stack example next? Like a blog, todo app, or auth system?
