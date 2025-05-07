Let's walk through **Full Stack Web Development in Rust**, focusing on building interactive web applications using **Leptos** — a modern, reactive web framework for Rust that supports both client-side (WebAssembly) and server-side rendering.

---

## 🚀 Why Rust + Leptos?

- **Rust**: Fast, safe systems programming language.
- **Leptos**: A declarative, reactive front-end framework written in Rust. Compiles to WASM for the browser or runs on the server with SSR/SSG/Hydration support.

---

# 🔧 Prerequisites

Before we begin:

- Basic knowledge of Rust
- Familiarity with HTML/CSS
- Understanding of async programming in Rust is helpful but not required

Install Rust if you haven’t already:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Make sure `wasm32-unknown-unknown` target is installed:
```bash
rustup target add wasm32-unknown-unknown
```

Install `cargo-leptos`:
```bash
cargo install cargo-leptos
```

---

# 📦 Project Setup

Create a new Leptos project:
```bash
cargo leptos new my_app
cd my_app
```

This creates a full-stack app structure with:

- `src/main.rs`: Server entry point
- `src/lib.rs`: Your component library
- `src/components.rs`: UI components
- `Cargo.toml`: Includes necessary features (`ssr`, `hydrate`, etc.)

---

# 🌐 Full Stack Structure Overview

Leptos supports multiple modes:

| Mode        | Description |
|-------------|-------------|
| Client-only (WASM) | Runs entirely in browser |
| Server-Side Rendering (SSR) | Rendered on server, then hydrated |
| Static Site Generation (SSG) | Pre-rendered at build time |

We’ll use **SSR + Hydration** mode — best for SEO and performance.

---

# 🧠 Core Concepts

### 1. Signals & Reactivity

Leptos uses **fine-grained reactivity** similar to SolidJS.

Example: Counter Component
```rust
use leptos::*;

#[component]
fn Counter() -> impl IntoView {
    let (count, set_count) = create_signal(0);

    view! {
        <div>
            <p>"Count: " { count }</p>
            <button on:click=move |_| set_count.update(|n| *n += 1)>"Increment"</button>
        </div>
    }
}
```

---

### 2. Components

Components are functions returning `impl IntoView`.

```rust
#[component]
fn Greeting(name: String) -> impl IntoView {
    view! {
        <h1>"Hello, " { name } "!"</h1>
    }
}
```

Use it like this:
```rust
view! {
    <Greeting name="Alice" />
}
```

---

### 3. Routing

Leptos has built-in routing via `leptos_router`.

Add routes in `main.rs`:
```rust
use leptos::*;
use leptos_router::*;

#[component]
fn App() -> impl IntoView {
    view! {
        <Router>
            <Routes>
                <Route path="/" view=HomePage />
                <Route path="/about" view=AboutPage />
            </Routes>
        </Router>
    }
}
```

Define route components:
```rust
#[component]
fn HomePage() -> impl IntoView {
    view! { <h1>"Home Page"</h1> }
}

#[component]
fn AboutPage() -> impl IntoView {
    view! { <h1>"About Us"</h1> }
}
```

---

### 4. Server-Side Data Fetching

You can define server functions that run only on the server (or be called from client via WASM RPC).

Example:
```rust
#[server(GetPostFn, "/api")]
async fn get_post(id: u32) -> Result<String, ServerFnError> {
    Ok(format!("Post #{}", id))
}
```

Call it from the client:
```rust
let post_resource = Resource::new(move || id.get(), move |id| async move {
    get_post(id).await.unwrap_or_default()
});
```

Then render:
```rust
{move || match post_resource.get() {
    Some(post) => view! { <p>{post}</p> }.into_any(),
    None => view! { <p>Loading...</p> }.into_any(),
}}
```

---

### 5. Styling

Leptos supports any CSS method:

- Global stylesheets
- TailwindCSS (recommended)
- CSS-in-RS libraries (like `leptos-style`)
- Inline styling with `style!` macro

Example:
```rust
style! { .my_button => "padding: 8px 16px; background: blue; color: white;" }

view! {
    <button class=my_button.on_click(...)>Click me</button>
}
```

---

### 6. Deployment

Build for production:
```bash
cargo leptos build
```

Output will be in `./dist`.

Deploy on platforms like:

- Vercel
- Netlify
- Cloudflare Workers
- Self-hosted with Axum backend

---

# 🧪 Example: To-Do List App

Let’s make a simple todo list:

```rust
#[derive(Clone)]
struct Todo {
    id: usize,
    text: String,
    completed: bool,
}

#[component]
fn TodoList() -> impl IntoView {
    let (todos, set_todos) = create_signal(vec![
        Todo { id: 0, text: "Learn Rust".to_string(), completed: false },
        Todo { id: 1, text: "Build Leptos app".to_string(), completed: true },
    ]);

    let add_todo = move |_| {
        let new_id = todos.with(|t| t.len());
        set_todos.update(|list| {
            list.push(Todo {
                id: new_id,
                text: format!("New Todo {}", new_id),
                completed: false,
            });
        });
    };

    view! {
        <div>
            <button on:click=add_todo>"Add Todo"</button>
            <ul>
                {move || todos.get().into_iter().map(|todo| {
                    view! {
                        <li>
                            <input type="checkbox" prop:checked=todo.completed />
                            <span class:red=move || todo.completed>"{todo.text}"</span>
                        </li>
                    }
                }).collect::<Vec<_>>()}
            </ul>
        </div>
    }
}
```

---

# 🛠️ Backend Integration (Axum)

Leptos integrates well with Axum for full-stack apps.

Add Axum dependencies in `Cargo.toml`:
```toml
[dependencies]
axum = "0.6"
tokio = { version = "1", features = ["full"] }
```

Create a basic API endpoint:
```rust
use axum::{response::Html, routing::get, Router};

async fn hello_world() -> Html<&'static str> {
    Html("<h1>Hello from Axum!</h1>")
}

#[tokio::main]
async fn main() {
    let app = Router::new().route("/", get(hello_world));
    axum::Server::bind(&"0.0.0.0:3000".parse().unwrap())
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

You can combine Axum + Leptos SSR seamlessly.

---

# 🧰 Tools You Might Want

- **TailwindCSS**: For styling
- **Trunk**: Build tool for WASM apps
- **Tauri / Wry**: Desktop apps using same frontend code
- **SSE/WebSockets**: Real-time updates
- **Leptos Query**: State management like React Query

---

# 📚 Resources

- [Leptos Official Book](https://leptos-rs.github.io/leptos/)
- [Leptos GitHub](https://github.com/leptos-rs/leptos)
- [Awesome Leptos](https://github.com/gbj/awesome-leptos)
- [Rust Language Docs](https://doc.rust-lang.org/book/)

---

# 🏁 Final Thoughts

Leptos gives you:

- **Blazing fast performance**
- **Type-safe interop between frontend and backend**
- **Reactive programming model**
- **Real full-stack capability**

With Rust, you write safer, faster code — and with Leptos, you bring that power to the web.

---

Would you like to build a **real-world app together** step-by-step? Like a blog engine, task manager, or chat app in Rust with Leptos?
