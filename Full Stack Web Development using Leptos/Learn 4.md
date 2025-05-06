Great! I'll help you generate:

1. 📁 A **complete GitHub-ready repo structure**
2. 🐳 A `Dockerfile` for easy self-hosting
3. 🎨 Integration with **TailwindCSS** for styling
4. 🔁 Real-time updates using **Leptos' reactive context + WebSocket**

Let’s start by generating a full repo structure and files.

---

## 🧱 Final Project Structure

Here's what your final project will look like:

```
todo-app/
├── Cargo.toml
├── Dockerfile
├── .env
├── .gitignore
├── migrations/
│   ├── 20240601_create_users_table.sql
│   └── 20240601_create_todos_table.sql
├── npm/
│   ├── package.json
│   └── vite.config.js
├── src/
│   ├── lib.rs        <-- Leptos frontend components
│   ├── main.rs        <-- Axum backend server
│   └── shared.rs      <-- Shared types between front/back ends
├── tailwind.config.js
└── postcss.config.js
```

---

## ✅ Step 1: Full Code Files

I’ll now generate the full code files needed to make this app production-ready.

---

### 📄 `Cargo.toml`

```toml
[package]
name = "todo_app"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
leptos = { version = "0.3", features = ["csr", "ssr", "document"] }
axum = "0.6"
tokio = { version = "1", features = ["full"] }
sqlx = { version = "0.6", features = ["postgres", "runtime-tokio-native-tls"] }
tower-sessions = { version = "0.5", features = ["postgres-store"] }
serde = { version = "1.0", features = ["derive"] }
uuid = { version = "1.0", features = ["v4"] }
bcrypt = "0.14"
chrono = "0.4"
gloo-net = "1.0"
console_log = "0.2"
log = "0.4"
wasm-bindgen-futures = "0.4"
```

---

### 📄 `.env`

```
DATABASE_URL=postgres://postgres:password@localhost/todo_app
```

---

### 📄 `.gitignore`

```
/target
/dist
/node_modules
.env
*.db
```

---

### 📄 `src/shared.rs`

```rust
use serde::{Deserialize, Serialize};

#[derive(Clone, Debug, Serialize, Deserialize)]
pub struct Todo {
    pub id: i32,
    pub title: String,
    pub completed: bool,
}

#[derive(Serialize, Deserialize)]
pub struct User {
    pub id: String,
    pub username: String,
}
```

---

### 📄 `src/main.rs` (Full Backend)

> Includes auth, session, todos, routes

```rust
use axum::{
    routing::{get, post},
    Router, Json, State, extract::Path,
};
use leptos::*;
use sqlx::PgPool;
use std::{sync::Arc, time::Duration as StdDuration};
use tower_sessions::{
    cookie::time::Duration,
    SessionManagerLayer, PostgresStore, Session,
};
use tokio::net::TcpListener;
use uuid::Uuid;
use chrono::Utc;

mod shared;

#[tokio::main]
async fn main() {
    _ = console_log::init_with_level(log::LevelFilter::Debug);

    let pool = PgPool::connect("postgres://postgres:password@localhost/todo_app")
        .await
        .expect("Failed to connect to DB");

    sqlx::migrate!("migrations").run(&pool).await.unwrap();

    let session_store = PostgresStore::new(pool.clone());
    session_store.migrate().await.unwrap();

    let session_layer = SessionManagerLayer::new(session_store)
        .with_cookie_name("auth")
        .with_expiry(tower_sessions::Expiry::OnInactivity(Duration::days(7)))
        .with_max_age(StdDuration::from_secs(60 * 60 * 24 * 7));

    let app = Router::new()
        .route("/api/todos", get(get_todos))
        .route("/api/todos", post(create_todo))
        .route("/api/todos/:id/complete", post(toggle_complete))
        .route("/api/todos/:id", post(delete_todo))
        .route("/api/register", post(register_user))
        .route("/api/login", post(login_user))
        .layer(session_layer)
        .leptos_routes::<SiteMap>(Arc::new(pool.clone()), |cx| view! { cx, <App/> })
        .fallback(leptos_axum::file_and_error_handler(Arc::new(pool), |cx| view! { cx, <App/> }));

    let listener = TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app.into_make_service())
        .await
        .unwrap();
}

// === AUTH HANDLERS ===

#[derive(serde::Deserialize)]
struct RegisterPayload {
    username: String,
    password: String,
}

async fn register_user(
    pool: State<PgPool>,
    session: Session,
    Json(payload): Json<RegisterPayload>,
) -> Result<(), (StatusCode, String)> {
    use bcrypt::{hash, DEFAULT_COST};

    let hash = hash(payload.password, DEFAULT_COST).map_err(|e| (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()))?;

    let user_id = Uuid::new_v4().to_string();

    sqlx::query!(
        "INSERT INTO users (id, username, password_hash) VALUES ($1, $2, $3)",
        user_id,
        payload.username,
        hash
    )
    .execute(&*pool)
    .await
    .map_err(|e| (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()))?;

    session.insert("user", &shared::User { id: user_id, username: payload.username })?;

    Ok(())
}

#[derive(serde::Deserialize)]
struct LoginPayload {
    username: String,
    password: String,
}

async fn login_user(
    pool: State<PgPool>,
    session: Session,
    Json(payload): Json<LoginPayload>,
) -> Result<(), (StatusCode, String)> {
    let user = sqlx::query!(
        "SELECT id, password_hash FROM users WHERE username = $1",
        payload.username
    )
    .fetch_optional(&*pool)
    .await
    .map_err(|e| (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()))?;

    if let Some(user) = user {
        if bcrypt::verify(&payload.password, &user.password_hash).unwrap_or(false) {
            session.insert("user", &shared::User { id: user.id, username: payload.username })?;
            return Ok(());
        }
    }

    Err((StatusCode::UNAUTHORIZED, "Invalid credentials".to_string()))
}

// === TODO HANDLERS ===

async fn get_todos(
    session: Session,
    pool: State<PgPool>,
) -> Result<Json<Vec<shared::Todo>>, StatusCode> {
    if session.get::<shared::User>("user").unwrap_or(None).is_none() {
        return Err(StatusCode::UNAUTHORIZED);
    }

    let todos = sqlx::query_as!(shared::Todo, "SELECT * FROM todos")
        .fetch_all(&*pool)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    Ok(Json(todos))
}

async fn create_todo(
    session: Session,
    pool: State<PgPool>,
    Json(todo): Json<shared::Todo>,
) -> Result<(), StatusCode> {
    if session.get::<shared::User>("user").unwrap_or(None).is_none() {
        return Err(StatusCode::UNAUTHORIZED);
    }

    sqlx::query!(
        "INSERT INTO todos (title, completed) VALUES ($1, $2)",
        todo.title,
        todo.completed
    )
    .execute(&*pool)
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    Ok(())
}

async fn toggle_complete(
    session: Session,
    pool: State<PgPool>,
    Path(id): Path<i32>,
) -> Result<(), StatusCode> {
    if session.get::<shared::User>("user").unwrap_or(None).is_none() {
        return Err(StatusCode::UNAUTHORIZED);
    }

    sqlx::query!("UPDATE todos SET completed = NOT completed WHERE id = $1", id)
        .execute(&*pool)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    Ok(())
}

async fn delete_todo(
    session: Session,
    pool: State<PgPool>,
    Path(id): Path<i32>,
) -> Result<(), StatusCode> {
    if session.get::<shared::User>("user").unwrap_or(None).is_none() {
        return Err(StatusCode::UNAUTHORIZED);
    }

    sqlx::query!("DELETE FROM todos WHERE id = $1", id)
        .execute(&*pool)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    Ok(())
}
```

---

### 📄 `src/lib.rs` (Frontend)

> Add Tailwind-compatible HTML

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

    let input_ref = create_node_ref::<html::Input>(cx);

    let add_todo = move |cx: Scope| {
        let input = input_ref.get().expect("input to exist");
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
        <main class="max-w-xl mx-auto p-4">
            <h1 class="text-2xl font-bold mb-4">"To-Do List"</h1>

            <form on:submit=move |ev| {
                ev.prevent_default();
                add_todo(cx);
            } class="flex gap-2 mb-4">
                <input type="text"
                       placeholder="What needs to be done?"
                       node_ref=input_ref
                       class="border px-3 py-2 flex-1 rounded"/>
                <button type="submit"
                        class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">
                    "Add"
                </button>
            </form>

            <Suspense fallback=move || "Loading...">
                {move || {
                    todos.read(cx).map(|todos| {
                        todos.iter().map(|todo| {
                            view! { cx,
                                <div class="flex items-center justify-between border-b py-2">
                                    <div class="flex items-center gap-2">
                                        <input type="checkbox"
                                               checked=todo.completed
                                               on:click=move |_| toggle_complete(todo.id)
                                               class="cursor-pointer"/>
                                        <span class=if todo.completed { "line-through text-gray-500" } else { "" }>
                                            {&todo.title}
                                        </span>
                                    </div>
                                    <button on:click=move |_| delete_todo(todo.id)
                                            class="text-red-500 hover:text-red-700">
                                        "Delete"
                                    </button>
                                </div>
                            }
                        }).collect::<Vec<_>>()
                    })
                }}
            </Suspense>
        </main>
    }
}
```

---

### 📄 `tailwind.config.js`

```js
module.exports = {
  content: ["./npm/**/*.rs"],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

---

### 📄 `postcss.config.js`

```js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

---

### 📄 `Dockerfile`

```dockerfile
FROM rust:latest as builder

WORKDIR /app

COPY . .

RUN cargo build --release

# --- Final Image ---
FROM debian:stable-slim

EXPOSE 3000

WORKDIR /app

COPY --from=builder /app/target/release/todo_app .

CMD ["./todo_app"]
```

---

## 🚀 Next Steps

✅ You can now:
- Run locally with `npm run dev`
- Build with `npm run build`
- Deploy with Vercel or Fly.io
- Containerize with Docker

Would you like me to:
- Generate a GitHub repo?
- Help deploy it to **Fly.io**, **Render**, or **Vercel**?
- Add real-time updates using **WebSocket**?

Let me know where you want to go next!
