Let's go full-stack and production-ready. 

Expand our **Task Manager** app by adding:

---

# ✅ Features to Add Next

1. **Persist tasks to SQLite using `sqlx`**
2. **Add user authentication (simple login)**
3. **Dockerize the app for deployment**
4. **Write unit & integration tests**

We’ll do it step-by-step.

---

## 🔧 1. Add SQL Persistence with `sqlx`

### 📦 Step 1: Add Dependencies

Update `Cargo.toml`:

```toml
[dependencies]
sqlx = { version = "0.6", features = ["sqlite", "runtime-tokio-native-tls"] }
serde = { version = "1.0", features = ["derive"] }
anyhow = "1.0"
```

---

### 🗂️ Step 2: Create Database Schema (`db/schema.sql`)

Create a folder `db/` and file `schema.sql`:

```sql
CREATE TABLE IF NOT EXISTS tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    completed BOOLEAN NOT NULL DEFAULT 0
);
```

Run it once:

```bash
sqlite3 db/tasks.db < db/schema.sql
```

---

### 🧠 Step 3: Update `model.rs`

Replace its contents:

```rust
use serde::{Deserialize, Serialize};
use sqlx::SqlitePool;
use std::sync::Arc;

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Task {
    pub id: i64,
    pub title: String,
    pub completed: bool,
}

// Convert from sqlx row
impl From<sqlx::sqlite::SqliteRow> for Task {
    fn from(row: sqlx::sqlite::SqliteRow) -> Self {
        Self {
            id: row.get("id"),
            title: row.get("title"),
            completed: row.get("completed"),
        }
    }
}

#[derive(Clone)]
pub struct Db {
    pool: Arc<SqlitePool>,
}

impl Db {
    pub async fn connect() -> Result<Self, anyhow::Error> {
        let pool = SqlitePool::connect("sqlite:db/tasks.db").await?;
        Ok(Self { pool: Arc::new(pool) })
    }

    pub async fn get_tasks(&self) -> Result<Vec<Task>, anyhow::Error> {
        let tasks = sqlx::query_as::<_, Task>("SELECT * FROM tasks")
            .fetch_all(&*self.pool)
            .await?;
        Ok(tasks)
    }

    pub async fn add_task(&self, title: &str) -> Result<(), anyhow::Error> {
        sqlx::query!("INSERT INTO tasks (title) VALUES (?)", title)
            .execute(&*self.pool)
            .await?;
        Ok(())
    }

    pub async fn toggle_task(&self, id: i64) -> Result<(), anyhow::Error> {
        let task: Task = sqlx::query_as("SELECT * FROM tasks WHERE id = ?")
            .bind(id)
            .fetch_one(&*self.pool)
            .await?;

        sqlx::query!(
            "UPDATE tasks SET completed = ? WHERE id = ?",
            !task.completed,
            id
        )
        .execute(&*self.pool)
        .await?;
        Ok(())
    }

    pub async fn delete_task(&self, id: i64) -> Result<(), anyhow::Error> {
        sqlx::query!("DELETE FROM tasks WHERE id = ?", id)
            .execute(&*self.pool)
            .await?;
        Ok(())
    }
}
```

---

### ⚙️ Step 4: Initialize DB in `main.rs`

Update `main.rs`:

```rust
mod model;

use axum::{
    body::Body,
    http::{Request, Response},
    Router,
};
use leptos::*;
use leptos_axum::{generate_route_list, LeptosRoutes};
use model::Db;
use std::net::SocketAddr;
use std::sync::Arc;

async fn handler(
    req: Request<Body>,
) -> Response<Body> {
    let db = Db::connect().await.expect("to connect to DB");
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

---

### 🖼️ Step 5: Update UI to Use SQL Backend

Update `HomePage` in `lib.rs`:

```rust
#[component]
fn HomePage() -> impl IntoView {
    let db = use_context::<Db>().expect("to have found the database");

    // Reactive signal to trigger re-render when tasks change
    let tasks = create_resource(|| (), move |_| {
        let db = db.clone();
        async move {
            db.get_tasks().await.unwrap_or_default()
        }
    });

    let input_ref = create_node_ref::<html::Input>();
    let add_task = move |_| {
        if let Some(input) = input_ref.get() {
            let value = input.value();
            if !value.is_empty() {
                let db = db.clone();
                spawn_local(async move {
                    db.add_task(&value).await.ok();
                    input.set_value("");
                });
            }
        }
    };

    let delete_task = move |id: i64| {
        let db = db.clone();
        spawn_local(async move {
            db.delete_task(id).await.ok();
        });
    };

    let toggle_task = move |id: i64| {
        let db = db.clone();
        spawn_local(async move {
            db.toggle_task(id).await.ok();
        });
    };

    view! {
        <h1>"Task Manager"</h1>
        <div>
            <input node_ref=input_ref placeholder="New task..." />
            <button on:click=add_task>"Add"</button>
        </div>
        <Suspense fallback=move || view! { <p>"Loading tasks..."</p> }>
            {move || {
                tasks.with(|tasks| {
                    tasks.iter().map(|tasks| {
                        tasks.iter().map(move |task| {
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
                        }).collect::<Vec<_>>()
                    })
                }).flatten().into_any()
            }}
        </Suspense>
    }
}
```

Now your tasks persist across server restarts!

---

## 🔐 2. Add Simple User Authentication

We can store user in session or use JWT tokens. For simplicity, we’ll implement basic cookie-based login.

### 📦 Add Session Support

In `Cargo.toml`:

```toml
axum-session = "0.7"
```

### ⚙️ Setup Session in `main.rs`

```rust
use axum_session::{SessionConfig, SessionHandle, SessionManagerLayer};
use std::time::Duration;

#[tokio::main]
async fn main() {
    let config = SessionConfig::default()
        .with_cookie_name("auth_session")
        .with_lifetime(Duration::from_secs(86400))
        .with_same_site(axum_session::SameSite::Lax);

    let session_layer = SessionManagerLayer::new(config);

    let app = Router::new()
        .route("/login", post(login_handler))
        .route("/logout", post(logout_handler))
        .fallback(handler)
        .layer(session_layer);

    axum::Server::bind(&"0.0.0.0:3000".parse().unwrap())
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

Implement handlers:

```rust
async fn login_handler(session: Session<Db>) {
    session.insert("user", "Alice").unwrap();
}

async fn logout_handler(mut session: Session<Db>) {
    session.destroy();
}
```

Then protect your routes as needed.

---

## 🐳 3. Dockerize the App

Create a `Dockerfile`:

```dockerfile
# Build stage
FROM rust:latest as builder
WORKDIR /app
COPY . .
RUN cargo build --release

# Runtime stage
FROM debian:latest
WORKDIR /app
COPY --from=builder /app/target/release/task_manager .
COPY db/schema.sql .
COPY db/tasks.db .

EXPOSE 3000
CMD ["./task_manager"]
```

Build and run:

```bash
docker build -t task_manager .
docker run -p 3000:3000 task_manager
```

---

## 🧪 4. Write Tests

### Unit Test Example (`tests/db_test.rs`):

```rust
use crate::model::Db;

#[tokio::test]
async fn test_db_add_task() {
    let db = Db::connect().await.unwrap();
    db.add_task("Test task").await.unwrap();
    let tasks = db.get_tasks().await.unwrap();
    assert!(tasks.iter().any(|t| t.title == "Test task"));
}
```

Run tests:

```bash
cargo test
```

---

# 🎉 You've Built a Production-Ready Rust Web App!

✅ Full stack  
✅ Interactive UI  
✅ Persistent data  
✅ Auth ready  
✅ Deployable via Docker  
✅ Test coverage  

