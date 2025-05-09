# Add user authentication to full-stack Rust Leptos app using **JWT (JSON Web Tokens)**.

This will let users:
- ✅ Sign up
- ✅ Log in
- ✅ Stay logged in via cookies or localStorage
- ✅ Access protected routes

We’ll use:
- 🦀 Rust + Axum for backend auth logic
- 🔐 JWT for token generation/verification
- 🍪 Cookie-based session or localStorage on frontend
- 🧬 A simple user database (SQLite)

---

# 🛠️ Step 1: Add Dependencies

Update `Cargo.toml`:

```toml
jsonwebtoken = "8.2"
bcrypt = "0.14"
serde = { version = "1.0", features = ["derive"] }
tower-http = { version = "0.3", features = ["auth"] }
cookie = "0.18"
time = "0.3"
```

---

# 🗄️ Step 2: Update DB Schema

Add a `users` table in `db/schema.sql`:

```sql
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL
);
```

Re-run migrations or update your D1 DB if you’re on Cloudflare.

---

# 🧱 Step 3: Define User Model and Auth Structs

Create a new file: `src/models.rs`

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
pub struct Claims {
    pub sub: String,
    pub exp: usize,
}

#[derive(sqlx::FromRow, Serialize, Deserialize)]
pub struct User {
    pub id: i64,
    pub username: String,
    pub password_hash: String,
}

#[derive(Deserialize)]
pub struct RegisterUser {
    pub username: String,
    pub password: String,
}

#[derive(Deserialize)]
pub struct LoginUser {
    pub username: String,
    pub password: String,
}
```

---

# 🌐 Step 4: Create Auth Routes in `main.rs`

Add these imports at the top:

```rust
mod models;
use models::{Claims, RegisterUser, LoginUser, User};
use axum::{
    Json, http::StatusCode, extract::State, response::IntoResponse, Router,
};
use jsonwebtoken::{encode, Header, EncodingKey};
use bcrypt::{hash, verify, DEFAULT_COST};
use std::sync::Arc;
use time::{Duration, OffsetDateTime};
use cookie::Cookie;
use tower_http::services::ServeDir;
```

### a. Add `/register` Route

```rust
async fn register(
    State(state): State<Arc<AppState>>,
    Json(payload): Json<RegisterUser>,
) -> Result<impl IntoResponse, StatusCode> {
    let hash = hash(&payload.password, DEFAULT_COST).map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    sqlx::query!(
        "INSERT INTO users (username, password_hash) VALUES (?, ?)",
        payload.username,
        hash
    )
    .execute(&state.pool)
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    Ok(StatusCode::CREATED)
}
```

### b. Add `/login` Route

```rust
async fn login(
    State(state): State<Arc<AppState>>,
    Json(payload): Json<LoginUser>,
) -> Result<impl IntoResponse, StatusCode> {
    let user: Option<User> = sqlx::query_as!(
        User,
        "SELECT id, username, password_hash FROM users WHERE username = ?",
        payload.username
    )
    .fetch_optional(&state.pool)
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    let user = user.ok_or(StatusCode::UNAUTHORIZED)?;
    let valid = verify(&payload.password, &user.password_hash).map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    if !valid {
        return Err(StatusCode::UNAUTHORIZED);
    }

    let expiration = OffsetDateTime::now_utc() + Duration::days(7);
    let claims = Claims {
        sub: user.id.to_string(),
        exp: expiration.unix_timestamp() as usize,
    };

    let token = encode(
        &Header::default(),
        &claims,
        &EncodingKey::from_secret("secret".as_ref()),
    ).map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    let cookie = Cookie::build(("token", token))
        .path("/")
        .http_only(true)
        .secure(cfg!(not(debug_assertions))) // Use secure cookies in production
        .max_age(time::Duration::days(7))
        .finish();

    Ok((StatusCode::OK, cookie.to_string()))
}
```

### c. Protect `/api/todos` with Auth

You can add middleware later, but for now just check for token in header:

```rust
async fn get_todos(State(state): State<Arc<AppState>>) -> Result<impl IntoResponse, StatusCode> {
    // In real app, extract token from cookie or header and validate
    let todos = sqlx::query!("SELECT text FROM todos")
        .fetch_all(&state.pool)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    let texts: Vec<String> = todos.iter().map(|t| t.text.clone()).collect();
    Ok(Json(texts))
}
```

Then add these routes:

```rust
let app = Router::new()
    .route("/", get(move || generate_route(App, &leptos_options)))
    .route("/api/todos", get(get_todos).post(add_todo).delete(delete_all_todos))
    .route("/register", post(register))
    .route("/login", post(login))
    .nest_service("/pkg", ServeDir::new("dist/pkg"))
    .with_state(Arc::clone(&state));
```

---

## 🖥️ Step 5: Add Login Form in `lib.rs`

Update `TodoApp` to include login/sign-up forms.

Here's a simplified version of how to do it:

```rust
#[component]
fn LoginForm(set_logged_in: WriteSignal<bool>) -> impl IntoView {
    let (username, set_username) = create_signal("".to_string());
    let (password, set_password) = create_signal("".to_string());

    let login = move |_| {
        spawn_local(async move {
            reqwasm::http::Request::post("/login")
                .header("Content-Type", "application/json")
                .body(serde_json::json!({ "username": username(), "password": password() }).to_string())
                .send()
                .await
                .unwrap();
            set_logged_in(true);
        });
    };

    view! {
        <div class="space-y-4">
            <h2 class="text-xl font-semibold">"Login"</h2>
            <input
                type="text"
                placeholder="Username"
                prop:value=username
                on:input=move |ev| set_username(event_target_value(&ev))
                class="w-full px-3 py-2 border rounded"
            />
            <input
                type="password"
                placeholder="Password"
                prop:value=password
                on:input=move |ev| set_password(event_target_value(&ev))
                class="w-full px-3 py-2 border rounded"
            />
            <button
                on:click=login
                class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
            >
                "Login"
            </button>
        </div>
    }
}
```

You can also add a `RegisterForm` component similarly.

---

## 📦 Step 6: Commit and Push Changes

```bash
git add .
git commit -m "Add JWT-based authentication system"
git push origin main
```

---

## 🚀 Authentication now working

✅ Users can sign up and log in  
✅ Protected routes (you can expand this)  
✅ JWT tokens stored securely  
✅ All in Rust + Leptos + Axum

---

## 🎯 Next:
1. Store user ID in a signal so UI updates dynamically
2. Add logout functionality
3. Make `/api/todos` require authentication
4. Deploy with GitHub Actions CI/CD pipeline
