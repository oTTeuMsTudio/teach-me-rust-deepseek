Great! Let's build a **Full Stack To-Do App using Leptos + Axum + PostgreSQL**.

This app will:
- Display a list of to-do items
- Allow users to add, mark complete, and delete items
- Persist data in a PostgreSQL database
- Use Rust on both frontend and backend 🚀

---

## 🧱 Step 1: Prerequisites

Make sure you have:

- [Rust installed](https://rustup.rs)
- `wasm-pack` and `cargo-leptos` installed:
```bash
cargo install wasm-pack cargo-leptos
```

- PostgreSQL running locally or remotely (use Docker if needed)

---

## 📦 Step 2: Create the Project

Generate a new fullstack Leptos project:

```bash
cargo leptos new todo_app --git
```

Choose the **Fullstack App (with Axum)** template.

Navigate into your project:

```bash
cd todo_app
npm install
```

---

## 🗄️ Step 3: Set Up the Database

### 1. Install SQLx CLI

```bash
cargo install sqlx-cli --features postgres
```

### 2. Create `.env` file

```env
DATABASE_URL=postgres://postgres:password@localhost/todo_app
```

> Replace with your actual DB credentials.

### 3. Create migration for to-do items

Run:

```bash
sqlx migrate add create_todos_table
```

Edit the generated migration in `migrations/xxxx_create_todos_table.sql`:

```sql
CREATE TABLE todos (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    completed BOOLEAN NOT NULL DEFAULT FALSE
);
```

Apply it:

```bash
sqlx migrate run
```

---

## 🧾 Step 4: Define Shared Types (`src/shared.rs`)

We’ll define our `Todo` struct that will be shared between frontend and backend.

```rust
use serde::{Deserialize, Serialize};

#[derive(Clone, Debug, Serialize, Deserialize)]
pub struct Todo {
    pub id: i32,
    pub title: String,
    pub completed: bool,
}
```

---

## 🏗️ Step 5: Build the Backend (`src/main.rs`)

Update `main.rs` to connect to the DB and expose API endpoints.

```rust
use axum::{
    routing::{get, post},
    Router,
};
use leptos::*;
use std::sync::Arc;
use tokio::net::TcpListener;
use sqlx::PgPool;
use todo_app::*;

#[tokio::main]
async fn main() {
    _ = console_log::init_with_level(log::LevelFilter::Debug);

    // Connect to DB
    let pool = PgPool::connect("postgres://postgres:password@localhost/todo_app")
        .await
        .expect("Failed to connect to DB");

    // Run migrations
    sqlx::migrate!("migrations").run(&pool).await.unwrap();

    // Setup routes
    let app = Router::new()
        .route("/api/todos", get(get_todos))
        .route("/api/todos", post(create_todo))
        .route("/api/todos/:id/complete", post(toggle_complete))
        .route("/api/todos/:id", post(delete_todo))
        .leptos_routes::<SiteMap>(Arc::new(pool.clone()), |cx| view! { cx, <App/> })
        .fallback(leptos_axum::file_and_error_handler(Arc::new(pool), |cx| view! { cx, <App/> }));

    let listener = TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app.into_make_service())
        .await
        .unwrap();
}

// === API HANDLERS ===

async fn get_todos(pool: axum::extract::State<PgPool>) -> impl axum::response::IntoResponse {
    use crate::shared::Todo;

    let todos = sqlx::query_as!(Todo, "SELECT * FROM todos")
        .fetch_all(&*pool)
        .await
        .unwrap();

    axum::Json(todos)
}

async fn create_todo(
    pool: axum::extract::State<PgPool>,
    axum::Json(payload): axum::Json<crate::shared::Todo>,
) {
    sqlx::query!(
        "INSERT INTO todos (title, completed) VALUES ($1, $2)",
        payload.title,
        payload.completed
    )
    .execute(&*pool)
    .await
    .unwrap();
}

async fn toggle_complete(
    pool: axum::extract::State<PgPool>,
    axum::extract::Path(id): axum::extract::Path<i32>,
) {
    sqlx::query!(
        "UPDATE todos SET completed = NOT completed WHERE id = $1",
        id
    )
    .execute(&*pool)
    .await
    .unwrap();
}

async fn delete_todo(
    pool: axum::extract::State<PgPool>,
    axum::extract::Path(id): axum::extract::Path<i32>,
) {
    sqlx::query!("DELETE FROM todos WHERE id = $1", id)
        .execute(&*pool)
        .await
        .unwrap();
}
```

---

## 💻 Step 6: Build the Frontend (`src/lib.rs`)

Replace contents of `src/lib.rs` with this interactive to-do UI:

```rust
use leptos::*;
use gloo_net::http::Request;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Clone, Debug)]
struct Todo {
    id: i32,
    title: String,
    completed: bool,
}

#[component]
fn App(cx: Scope) -> impl IntoView {
    let todos = create_resource(cx, || (), |_| async move {
        Request::get("/api/todos")
            .send()
            .await
            .unwrap()
            .json::<Vec<Todo>>()
            .await
            .unwrap_or_default()
    });

    let add_todo = move |cx: Scope, input: NodeRef<html::Input>| {
        let input = input.get().expect("input to exist");
        let value = input.value();
        if value.is_empty() {
            return;
        }

        spawn_local(async move {
            #[derive(Serialize)]
            struct Payload {
                title: String,
                completed: bool,
            }

            Request::post("/api/todos")
                .json(&Payload {
                    title: value,
                    completed: false,
                })
                .unwrap()
                .send()
                .await
                .unwrap();

            todos.refetch();
        });

        input.set_value("");
    };

    let toggle_complete = move |id: i32| {
        spawn_local(async move {
            Request::post(&format!("/api/todos/{}/complete", id))
                .send()
                .await
                .unwrap();
            todos.refetch();
        });
    };

    let delete_todo = move |id: i32| {
        spawn_local(async move {
            Request::post(&format!("/api/todos/{}/delete", id))
                .send()
                .await
                .unwrap();
            todos.refetch();
        });
    };

    view! { cx,
        <main>
            <h1>"To-Do List"</h1>

            <form on:submit=move |ev| {
                ev.prevent_default();
                add_todo(cx, input_ref)
            }>
                <input type="text" placeholder="What needs to be done?" node_ref=input_ref/>
                <button type="submit">"Add"</button>
            </form>

            <Suspense fallback=move || "Loading...">
                {move || {
                    todos.read(cx).map(|todos| {
                        todos.iter().map(|todo| {
                            view! { cx,
                                <div class="todo">
                                    <input
                                        type="checkbox"
                                        checked=todo.completed
                                        on:click=move |_| toggle_complete(todo.id)
                                    />
                                    <span style:color=if todo.completed { "gray" } else { "black" }>
                                        {&todo.title}
                                    </span>
                                    <button on:click=move |_| delete_todo(todo.id)>"Delete"</button>
                                </div>
                            }
                        }).collect::<Vec<_>>()
                    })
                }}
            </Suspense>
        </main>
    }
}

let input_ref = create_node_ref::<html::Input>(cx);
```

---

## ✅ Step 7: Run Your App

In the terminal:

```bash
npm run dev
```

Open your browser at http://localhost:3000 — you should see your to-do app!

---

## 🚀 Bonus Features You Can Add Later

- Edit existing to-dos
- Filter by completed / active
- Persist state with localStorage or sessions
- Add authentication
- Style with TailwindCSS or Leptos-Stylist

---

## 🎉 Congratulations!

You've built a **full-stack web application in Rust using Leptos and Axum**, backed by PostgreSQL.

Would you like me to help you deploy this to production? Or maybe extend it with user authentication or real-time updates using WebSocket?
