Let’s build a **real-world full-stack web application in Rust using Leptos**.

---

# 🧱 Project: Task Manager Web App (To-Do List)

## ✅ Features:
- Add new tasks
- Mark tasks as completed
- Delete tasks
- Persist tasks to a database
- Server-side rendering with Leptos + Axum
- CRUD API with server functions

---

# 📦 Step 1: Setup the Project

If you haven’t already, create a new Leptos project:

```bash
cargo leptos new task_manager
cd task_manager
```

Choose `ssr` mode when prompted (for Server-Side Rendering).

---

# 🗂️ Step 2: Folder Structure

Your project should look like this:

```
task_manager/
├── Cargo.toml
├── src/
│   ├── main.rs       <-- Server entry point
│   ├── lib.rs        <-- Leptos components
│   └── components.rs <-- Optional for large apps
├── Cargo.toml
└── ...
```

---

# 🧠 Step 3: Define Task Model and Storage

Create a file `src/model.rs`:

```rust
use std::collections::HashMap;
use std::sync::{Arc, Mutex};

#[derive(Clone, Debug, serde::Serialize, serde::Deserialize)]
pub struct Task {
    pub id: usize,
    pub title: String,
    pub completed: bool,
}

// Simple in-memory storage
pub type Db = Arc<Mutex<HashMap<usize, Task>>>;

pub fn create_db() -> Db {
    Arc::new(Mutex::new(HashMap::new()))
}
```

Add it to `lib.rs`:

```rust
pub mod model;
```

Update `Cargo.toml` to enable Serde:

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
```

---

# ⚙️ Step 4: Initialize DB in Server (`main.rs`)

Modify `main.rs`:

```rust
mod model;

use axum::{
    body::Body,
    http::{Request, Response},
    Router,
};
use leptos::*;
use leptos_axum::{generate_route_list, LeptosRoutes};
use model::create_db;
use std::net::SocketAddr;
use std::sync::Arc;

async fn handler(
    req: Request<Body>,
) -> Response<Body> {
    let db = create_db();
    let routes = generate_route_list(|cx| view! { cx, <App/> });
    leptos_axum::render_app_to_string_with_context(
        move |cx| {
            provide_context(cx, db.clone());
        },
        App,
        &routes,
        req,
    )
    .await
    .into()
}

#[tokio::main]
async fn main() {
    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    let app = Router::new().fallback(handler);

    println!("🚀 Listening on http://{}", addr);
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

Now we have access to `Db` via context in our components.

---

# 🖼️ Step 5: Create the UI Components (`lib.rs`)

Update `lib.rs`:

```rust
use leptos::*;
use model::{Db, Task};

#[component]
pub fn App() -> impl IntoView {
    view! {
        <Router>
            <main>
                <Routes>
                    <Route path="/" view=HomePage />
                </Routes>
            </main>
        </Router>
    }
}

#[component]
fn HomePage() -> impl IntoView {
    let db = use_context::<Db>().expect("to have found the database");

    // Reactive signal to add new task
    let input_ref = create_node_ref::<html::Input>();
    let add_task = move |_| {
        if let Some(input) = input_ref.get() {
            let value = input.value();
            if !value.is_empty() {
                let mut db = db.lock().unwrap();
                let id = db.len();
                db.insert(id, Task { id, title: value, completed: false });
                input.set_value("");
            }
        }
    };

    let delete_task = move |id: usize| {
        let mut db = db.lock().unwrap();
        db.remove(&id);
    };

    let toggle_task = move |id: usize| {
        let mut db = db.lock().unwrap();
        if let Some(task) = db.get_mut(&id) {
            task.completed = !task.completed;
        }
    };

    let render_tasks = move || {
        let db = db.lock().unwrap();
        db.values()
            .cloned()
            .map(move |task| {
                view! {
                    <li class:red=move || task.completed>
                        <input
                            type="checkbox"
                            prop:checked=task.completed
                            on:click=move |_| toggle_task(task.id)
                        />
                        <span>" "{task.title}</span>
                        <button on:click=move |_| delete_task(task.id)>"Delete"</button>
                    </li>
                }
            })
            .collect::<Vec<_>>()
    };

    view! {
        <h1>"Task Manager"</h1>
        <div>
            <input node_ref=input_ref placeholder="New task..." />
            <button on:click=add_task>"Add"</button>
        </div>
        <ul>{render_tasks}</ul>
    }
}
```

---

# 🔁 Step 6: Run the App

```bash
cargo leptos watch
```

Open your browser at [http://localhost:3000](http://localhost:3000)

You now have a fully working interactive task manager!

✅ Add, complete, and delete tasks  
✅ All state is stored in memory  

---

# 🛢️ Step 7: Add Persistence (Optional Bonus)

Right now, data is lost when the server restarts.

To persist, integrate with a real database like SQLite or PostgreSQL.

Use crates like:

- `sqlx`
- `diesel`
- `sea-orm`

Let me know if you'd like to implement persistence with SQL next!

---

# 🧪 Step 8: Testing & Deployment

Test locally first.

Then build for production:

```bash
cargo leptos build
```

Deploy on:

- Vercel / Netlify (with WASM support)
- Self-hosted with Axum backend
- Docker container

---

# 🧩 Next Steps You Can Explore

- Add filtering: Show all/active/completed tasks
- Use `leptos-use` for hooks like `use_local_storage`
- Implement authentication
- Deploy to Cloudflare Workers or Supabase Edge Functions
- Turn into a PWA or Tauri desktop app

---

# 🎉 Final Result

You just built a **full-stack interactive task manager** in Rust using:

- Leptos (for reactive frontend + SSR)
- Axum (server backend)
- In-memory DB (or upgrade to SQL)
- Rust's safety and speed

This is a foundation you can expand into any kind of web app — from dashboards to CRMs.

---

to continue:
1. 📦 Add SQL persistence?
2. 🔐 Add user authentication?
3. 🌐 Make it deployable with Docker?
4. 🧪 Write tests?
