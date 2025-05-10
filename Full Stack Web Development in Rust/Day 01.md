# Real-world full-stack app using Leptos and Postgres.

We'll build a simple **Todo List App** with:
- ✅ Frontend: Leptos (Rust + WASM)
- ✅ Backend: Axum (Rust HTTP server)
- ✅ Database: PostgreSQL
- ✅ ORM: SQLx
- ✅ Server Functions: `#[server]` to interact with the DB

---

# 🧱 Full-Stack Todo App in Rust with Leptos, Axum & Postgres

## 🗂️ Project Structure Recap

```
my_todo_app/
├── Cargo.toml
├── crates/
│   ├── my_todo_app/          # frontend logic
│   └── my_todo_app_axum/     # backend logic (Axum server)
├── migrations/               # SQLx migration files
├── index.html
└── ...
```

---

## 1. 🔧 Setup Dependencies

Add these to your root `Cargo.toml`:

```toml
[workspace]
members = [
    "crates/my_todo_app",
    "crates/my_todo_app_axum"
]

[dependencies]
sqlx = { version = "0.6", features = ["postgres", "runtime-tokio-native-tls"] }
serde = { version = "1.0", features = ["derive"] }
```

Also update the `my_todo_app_axum/Cargo.toml` to include dependencies:

```toml
[dependencies]
axum = "0.6"
leptos = { path = "../../crates/my_todo_app" }
leptos_config = "0.1"
leptos_router = "0.1"
tokio = { version = "1", features = ["full"] }
sqlx = { version = "0.6", features = ["postgres", "runtime-tokio-native-tls"] }
serde = { version = "1.0", features = ["derive"] }
dotenvy = "0.15"
```

---

## 2. 🛠️ Set Up Postgres & Migrations

### Create `.env` file at root:

```env
DATABASE_URL=postgres://postgres:password@localhost:5432/todo?sslmode=disable
```

### Create a database and run migrations

```bash
createdb todo
```

Create a folder for migrations:

```bash
mkdir -p migrations
```

Create `migrations/0001_initial.sql`:

```sql
CREATE TABLE IF NOT EXISTS todos (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    completed BOOLEAN NOT NULL DEFAULT FALSE
);
```

Now you can apply it via SQLx CLI:

```bash
cargo install sqlx-cli
sqlx migrate add init
sqlx migrate run
```

> Make sure Postgres is running locally (`brew services start postgresql` on macOS)

---

## 3. 📦 Define Todo Model

In `crates/my_todo_app/src/lib.rs`, define a shared model:

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Clone, Debug)]
pub struct Todo {
    pub id: i32,
    pub title: String,
    pub completed: bool,
}
```

---

## 4. 🧠 Server Functions (CRUD)

Still in `lib.rs`, define server functions:

```rust
use leptos::*;
use sqlx::PgPool;
use crate::Todo;

// Initialize DB pool from context
fn get_pool() -> Result<PgPool, ServerFnError> {
    use_context::<PgPool>().ok_or_else(|| ServerFnError::new("Could not get DB pool"))
}

#[server(CreateTodo)]
pub async fn create_todo(title: String) -> Result<Vec<Todo>, ServerFnError> {
    let pool = get_pool()?;
    sqlx::query_as!(
        Todo,
        r#"INSERT INTO todos (title) VALUES ($1) RETURNING *"#,
        title
    )
    .fetch_all(&pool)
    .await
    .map_err(|e| ServerFnError::new(e.to_string()))
}

#[server(DeleteTodo)]
pub async fn delete_todo(id: i32) -> Result<Vec<Todo>, ServerFnError> {
    let pool = get_pool()?;
    sqlx::query_as!(
        Todo,
        r#"DELETE FROM todos WHERE id = $1 RETURNING *"#,
        id
    )
    .fetch_all(&pool)
    .await
    .map_err(|e| ServerFnError::new(e.to_string()))
}

#[server(ToggleTodo)]
pub async fn toggle_todo(id: i32) -> Result<Vec<Todo>, ServerFnError> {
    let pool = get_pool()?;
    sqlx::query_as!(
        Todo,
        r#"UPDATE todos SET completed = NOT completed WHERE id = $1 RETURNING *"#,
        id
    )
    .fetch_all(&pool)
    .await
    .map_err(|e| ServerFnError::new(e.to_string()))
}

#[server(GetTodos)]
pub async fn get_todos() -> Result<Vec<Todo>, ServerFnError> {
    let pool = get_pool()?;
    sqlx::query_as!(Todo, r#"SELECT * FROM todos"#)
        .fetch_all(&pool)
        .await
        .map_err(|e| ServerFnError::new(e.to_string()))
}
```

---

## 5. 🌐 Inject DB Pool into Axum Server

Update `crates/my_todo_app_axum/src/main.rs`:

```rust
use axum::{Router, routing::get};
use leptos::*;
use leptos_config::get_configuration;
use my_todo_app::*;
use sqlx::PgPool;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    // Load config
    let conf = get_configuration(Some("Cargo.toml")).unwrap();
    let leptos_options = Arc::new(conf.leptos_options);

    // Connect to DB
    let db_url = std::env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    let pool = PgPool::connect(&db_url).await.unwrap();

    // Add pool to context
    let pool_arc = Arc::new(pool);
    provide_context_in_leptos_runtime(leptos_options.clone(), pool_arc);

    // Build routes
    let routes = generate_route_list(App);

    let app = Router::new()
        .leptos_routes(&leptos_options, routes, App)
        .fallback(file_and_error_handler(App));

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("🚀 Listening on http://localhost:3000");
    axum::serve(listener, app.into_make_service()).await.unwrap();
}

// Helper to inject DB pool into Leptos runtime
fn provide_context_in_leptos_runtime(
    leptos_options: Arc<LeptosOptions>,
    pool: Arc<PgPool>,
) {
    let rt = Runtime::new().unwrap();
    rt.spawn(async move {
        _ = leptos::mount_to_body(move |cx| {
            provide_context(cx, pool.clone());
            view! { cx, <App/> }
        });
    });
}
```

---

## 6. 🖥️ Build UI (Frontend Component)

Update `crates/my_todo_app/src/lib.rs` inside `App()`:

```rust
#[component]
pub fn App() -> impl IntoView {
    let get_todos_action = create_server_action::<GetTodos>();
    let create_todo_action = create_server_action::<CreateTodo>();
    let delete_todo_action = create_server_action::<DeleteTodo>();
    let toggle_todo_action = create_server_action::<ToggleTodo>();

    let todos = move || {
        get_todos_action.value().get().unwrap_or_default()
    };

    view! {
        <div>
            <h1>"Rust Todo App"</h1>

            <form on:submit=move |ev| {
                ev.prevent_default();
                let title = event_target_value(&ev, "title");
                create_todo_action.dispatch(CreateTodo { title });
            }>
                <input type="text" name="title" placeholder="New task..." />
                <button type="submit">"Add"</button>
            </form>

            <ul>
                {move || todos().into_iter().map(|todo| {
                    let delete_id = todo.id;
                    let toggle_id = todo.id;
                    view! {
                        <li style:color=if todo.completed { "gray" } else { "black" }>
                            <input type="checkbox"
                                checked=todo.completed
                                on:click=move |_| {
                                    toggle_todo_action.dispatch(ToggleTodo { id: toggle_id })
                                }
                            />
                            {todo.title}
                            <button on:click=move |_| {
                                delete_todo_action.dispatch(DeleteTodo { id: delete_id })
                            }>"X"</button>
                        </li>
                    }
                }).collect::<Vec<_>>()}
            </ul>
        </div>
    }
}
```

---

## 7. 🚀 Run The App

Start the dev server:

```bash
trunk serve
```

Open [http://localhost:3000](http://localhost:3000)

Now should see a working todo list that:
- Adds new tasks
- Toggles completion status
- Deletes tasks
- Persists all changes in Postgres

---

## 🎉 Full-Stack Rust App now working:

✅ Used Leptos for interactive UI  
✅ Used Axum as backend  
✅ Connected to PostgreSQL  
✅ Used SQLx for queries  
✅ Used `#[server]` functions for CRUD  
✅ Fully typed and safe

---

## 🧠 Going further:
- ✅ Add filtering: show All / Active / Completed
- ✅ Add user authentication (JWT or OAuth)
- ✅ Deploy to Cloudflare Workers / Vercel / AWS Lambda Rust
- ✅ Use Diesel instead of SQLx
- ✅ Add real-time updates with WebSocket
