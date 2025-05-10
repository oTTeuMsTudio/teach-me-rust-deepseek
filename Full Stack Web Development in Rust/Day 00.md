# Full-stack web development in Rust using Leptos, a modern, reactive framework for building front-end and full-stack web apps entirely in Rust.

This guide will cover:

1. 🧠 What is Leptos?
2. 🛠️ Setting up your environment
3. 🏗️ Building a frontend app with Leptos
4. 🌐 Adding server functions (server-side rendering / API)
5. 📦 Full-stack architecture overview
6. 🚀 Deployment

---

## 1. 💡 What is Leptos?

Leptos is a **Rust web framework** that lets you build **reactive UIs** using Rust syntax (not JS or TS). It compiles to WebAssembly (WASM), allowing you to write both the **frontend** and **backend** in Rust.

### Features:
- Reactive programming model like React/SolidJS
- Works in WASM (client-side) and SSR/hydration (server-side)
- Server functions: call backend logic directly from frontend code
- No JavaScript required (but interoperability possible)

---

## 2. 🛠️ Setup Your Environment

### Prerequisites

Install Rust:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Install `wasm-pack`:
```bash
cargo install wasm-pack
```

Install `trunk` (build tool for Leptos):
```bash
cargo install trunk
```

Install `cargo-leptos`:
```bash
cargo install cargo-leptos
```

---

## 3. 🏗️ Create a New Leptos Project

Create a new project:
```bash
cargo leptos new my_app
cd my_app
```

Choose template:
```
Which flavor of Leptos would you like?
1. CSR (Client Side Rendering)
2. SSR (Server Side Rendering + Hydration)
3. Both (Fullstack with Trunk + Axum backend)
```

👉 Choose **3. Both** for full-stack capabilities.

Now project structure looks like this:

```
my_app/
├── Cargo.toml
├── crates/
│   ├── my_app/
│   └── my_app_axum/  # server-side part
├── Cargo.toml
├── target/
├── index.html
└── ...
```

---

## 4. 🌐 Add Frontend & Backend Logic

### 4.1 Frontend (Leptos Component)

Edit: `crates/my_app/src/lib.rs`

```rust
use leptos::*;

#[component]
pub fn App() -> impl IntoView {
    let (count, set_count) = create_signal(0);

    view! {
        <main>
            <h1>"Welcome to Leptos!"</h1>
            <button on:click=move |_| set_count.update(|n| *n += 1)>
                "Click me: "
                {count}
            </button>
        </main>
    }
}
```

### 4.2 Backend (Axum Server)

Edit: `crates/my_app_axum/src/main.rs`

```rust
use axum::{routing::get, Router};
use leptos::*;
use leptos_config::get_configuration;
use my_app::*;

#[tokio::main]
async fn main() {
    // Get config
    let conf = get_configuration(Some("Cargo.toml")).unwrap();
    let leptos_options = conf.leptos_options;

    // Build handler for Leptos app
    let routes = generate_route_list(App);

    // Start Axum server
    let app = Router::new()
        .route("/api/increment", get(increment))
        .leptos_routes(&leptos_options, routes, App)
        .fallback(file_and_error_handler(App));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("🚀 Listening on http://localhost:3000");
    axum::serve(listener, app.into_make_service())
        .await
        .unwrap();
}

// Simple API endpoint
async fn increment() -> String {
    format!("Incremented at {}", chrono::Utc::now())
}
```

### 4.3 Call Server Function from Frontend

Leptos supports calling server functions directly from the client via WASM.

Add this inside your `App` component:

```rust
use leptos_meta::*;
use leptos_router::*;

#[server(IncreaseCount)]
async fn increase_count(count: i32) -> Result<String, ServerFnError> {
    Ok(format!("New count: {}", count + 1))
}

#[component]
pub fn App() -> impl IntoView {
    let (count, set_count) = create_signal(0);
    let action = create_server_action::<IncreaseCount>();

    view! {
        <main>
            <h1>"Counter"</h1>
            <p>{move || count.get()}</p>

            <ActionForm action>
                <input type="number" name="count" prop:value=count />
                <input type="submit" value="Increase on Server" />
            </ActionForm>

            <p><b>"Result from server:"</b> {move || action.value().get()}</p>
        </main>
    }
}
```

This shows how we can:
- Send data from the client to the server
- Run logic on the server
- Return result to the client reactivity system

---

## 5. 📦 Full-Stack Architecture Overview

Here’s what a typical Leptos full-stack app includes:

| Layer         | Tech Used                  |
|---------------|----------------------------|
| Frontend      | Leptos + Rust              |
| Build Tool    | Trunk                      |
| Backend       | Axum (Rust HTTP server)    |
| Templates     | HTML + Leptos views        |
| Routing       | Leptos Router              |
| State Mgmt    | Signals, Memo, Resources   |
| Server Fns    | `#[server]`, Actions       |
| DB Access     | SQLx, Diesel, etc.         |

Now can connect to databases like PostgreSQL or SQLite using libraries like `sqlx`.

---

## 6. 🚀 Deploying Your App

### Option 1: Local Dev
```bash
trunk serve
```

### Option 2: Production Build
```bash
trunk build --release
```

Now static assets are in `dist/`. Now can deploy them to any static hosting service (e.g., Vercel, Netlify, Cloudflare Pages).

But since using an Axum backend, need a Rust-capable runtime or container support.

### Option 3: Dockerize

Use Docker to run your Rust backend (Axum + Leptos):

```Dockerfile
FROM rust:latest as builder
WORKDIR /app
COPY . .
RUN cargo build --release

FROM gcr.io/distroless/cc-debian11
COPY --from=builder /app/target/release/my_app_axum /
CMD ["/my_app_axum"]
```

---

## 📚 Learning Resources

- 📘 [Leptos Docs](https://docs.rs/leptos/latest/leptos/)
- 📺 [Leptos YouTube Tutorials](https://www.youtube.com/results?search_query=leptos+rust)
- 🧪 [Leptos Examples](https://github.com/leptos-rs/leptos/tree/main/examples)
- 🛠️ [Trunk](https://trunkrs.dev/)
- 🧱 [Leptos Starter Template](https://github.com/leptos-rs/start)

---

## ✅ Summary Checklist

✅ Rust installed  
✅ `wasm-pack`, `trunk`, `cargo-leptos` installed  
✅ Created full-stack Leptos project  
✅ Built interactive frontend with signals  
✅ Wrote server function (`#[server]`)  
✅ Called server function from frontend  
✅ Built and deployed app  
