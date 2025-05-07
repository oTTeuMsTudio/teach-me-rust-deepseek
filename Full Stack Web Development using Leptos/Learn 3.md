Let's take your **To-Do App** to the next level by:

1. ✅ Adding **User Authentication**
2. 🚀 Deploying it to **production using Vercel + Cloudflare Workers**

We’ll walk through both in this guide.

---

## 🔐 Part 1: Add User Authentication

We'll implement:
- Login / Register
- Protected routes
- Session management with `tower-sessions`

---

### Step 1: Add Dependencies (`Cargo.toml`)

```toml
[dependencies]
tower-sessions = "0.5"
tower-sessions-sqlx-store = "0.1"
uuid = { version = "1.0", features = ["v4"] }
chrono = "0.4"
```

---

### Step 2: Create Users Table

Run migration:

```bash
sqlx migrate add create_users_table
```

Edit the migration:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    username TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

Apply:

```bash
sqlx migrate run
```

---

### Step 3: Update Shared Types (`src/shared.rs`)

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

### Step 4: Setup Sessions and Auth Middleware

In `main.rs`, update imports:

```rust
use tower_sessions::{
    cookie::time::Duration,
    sqlx::PgPool,
    Expiry, SessionManagerLayer, PostgresStore,
};
use std::time::Duration as StdDuration;
use uuid::Uuid;
use chrono::Utc;
```

Add session store setup:

```rust
let pool = PgPool::connect("postgres://postgres:password@localhost/todo_app")
    .await
    .expect("Failed to connect to DB");

// Setup session store
let session_store = PostgresStore::new(pool.clone());
session_store.migrate().await.unwrap();

let session_layer = SessionManagerLayer::new(session_store)
    .with_cookie_name("auth")
    .with_expiry(Expiry::OnInactivity(Duration::days(7)))
    .with_max_age(StdDuration::from_secs(60 * 60 * 24 * 7));
```

Now wrap your router:

```rust
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
```

---

### Step 5: Add Auth Handlers

```rust
async fn register_user(
    pool: State<PgPool>,
    session: Session,
    Json(payload): Json<serde_json::Value>,
) -> Result<impl IntoResponse, (StatusCode, String)> {
    let username = payload["username"]
        .as_str()
        .ok_or((StatusCode::BAD_REQUEST, "Missing username".to_string()))?;
    let password = payload["password"]
        .as_str()
        .ok_or((StatusCode::BAD_REQUEST, "Missing password".to_string()))?;

    let password_hash = bcrypt::hash(password, bcrypt::DEFAULT_COST).unwrap();

    let user_id = Uuid::new_v4().to_string();

    sqlx::query!(
        "INSERT INTO users (id, username, password_hash) VALUES ($1, $2, $3)",
        user_id,
        username,
        password_hash
    )
    .execute(&*pool)
    .await
    .map_err(|e| (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()))?;

    session.insert("user", &User { id: user_id, username: username.to_string() })?;

    Ok(StatusCode::OK)
}

async fn login_user(
    pool: State<PgPool>,
    session: Session,
    Json(payload): Json<serde_json::Value>,
) -> Result<impl IntoResponse, (StatusCode, String)> {
    let username = payload["username"]
        .as_str()
        .ok_or((StatusCode::BAD_REQUEST, "Missing username".to_string()))?;
    let password = payload["password"]
        .as_str()
        .ok_or((StatusCode::BAD_REQUEST, "Missing password".to_string()))?;

    let user = sqlx::query!(
        "SELECT id, password_hash FROM users WHERE username = $1",
        username
    )
    .fetch_optional(&*pool)
    .await
    .map_err(|e| (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()))?;

    if let Some(user) = user {
        if bcrypt::verify(password, &user.password_hash).unwrap_or(false) {
            session.insert("user", &User { id: user.id, username: username.to_string() })?;
            return Ok(StatusCode::OK);
        }
    }

    Err((StatusCode::UNAUTHORIZED, "Invalid credentials".to_string()))
}
```

---

### Step 6: Protect Routes

Update your todo endpoints to require auth:

```rust
async fn get_todos(
    session: Session,
    pool: State<PgPool>,
) -> Result<Json<Vec<Todo>>, StatusCode> {
    if session.get::<User>("user").unwrap_or(None).is_none() {
        return Err(StatusCode::UNAUTHORIZED);
    }

    let todos = sqlx::query_as!(Todo, "SELECT * FROM todos")
        .fetch_all(&*pool)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    Ok(Json(todos))
}
```

Repeat for other routes.

---

### Step 7: Add Login UI in Frontend (`src/lib.rs`)

Add a login form and protect the todo list:

```rust
#[component]
fn App(cx: Scope) -> impl IntoView {
    let login_action = create_server_action::<Login>(cx);

    view! { cx,
        <main>
            <ActionForm action=login_action>
                <input type="text" name="username" placeholder="Username"/>
                <input type="password" name="password" placeholder="Password"/>
                <button type="submit">"Login"</button>
            </ActionForm>
        </main>
    }
}
```

And define the server action:

```rust
#[server(Login, "/api")]
async fn login(username: String, password: String) -> Result<(), ServerFnError> {
    // Call backend API via fetch or leptos-meta
    Ok(())
}
```

---

## ☁️ Part 2: Deploy Your App

We’ll deploy:
- **Frontend**: Static assets on **Vercel**
- **Backend**: Rust Axum server on **Cloudflare Workers + D1 Database**

> Note: Cloudflare D1 doesn't support PostgreSQL yet — you can use SQLite instead or host Postgres elsewhere (e.g., Neon.tech)

---

### Option A: Use Neon.tech for Postgres

1. Go to [https://neon.tech](https://neon.tech)
2. Create a new project and get connection string
3. Update `.env`:

```env
DATABASE_URL=postgres://your-neon-db-url
```

---

### Option B: Use SQLite Locally (for demo)

Use `sqlx` with SQLite for deployment simplicity.

Update dependencies:

```toml
sqlx = { version = "0.6", features = ["sqlite", "runtime-tokio-native-tls"] }
```

Set up database:

```bash
touch todo.db
sqlx migrate run --database-url sqlite:todo.db
```

Update main.rs:

```rust
let pool = SqlitePool::connect("sqlite:todo.db").await.unwrap();
```

---

### Deploy to Vercel + Cloudflare Workers

#### 1. Backend: Build Worker

Install Wrangler:

```bash
npm install -g wrangler
```

Create a `worker` folder with a basic Rust worker that runs Axum.

Check out Leptos' official examples for deploying to workers:
- https://github.com/gbj/leptos/tree/main/examples

You may need to adapt the runtime slightly.

#### 2. Frontend: Deploy with Vercel

Just run:

```bash
npm run build
vercel dist
```

---

## 🧠 Summary

✅ You’ve added:
- User registration and login
- Session-based authentication
- Protected routes
- Ready-to-deploy architecture

🚀 You can now:
- Deploy frontend to **Vercel**
- Deploy backend to **Cloudflare Workers**, **Fly.io**, or **Render**
- Use **Neon.tech** for hosted Postgres

---

## 📦 Want More?

Would you like me to generate:
- The full GitHub repo?
- A Dockerfile for self-hosting?
- Integration with TailwindCSS?
- Real-time updates using WebSocket or Sycamore Live?

Let me know what you'd like next!
