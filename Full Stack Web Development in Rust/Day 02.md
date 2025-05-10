# Taking full-stack Leptos + Axum + Postgres Todo App to the next level by:

✅ Adding **user authentication**  
✅ Using **JWT (JSON Web Tokens)** for login/session management  
✅ Protecting server functions with middleware  
✅ Storing users in Postgres

---

# 🔐 Leptos Full-Stack Auth: Add Login & Protected Routes

## 🧩 Features We'll Build
- Register and Login forms
- JWT-based session tokens
- User table in Postgres
- Protected routes (e.g., only logged-in users can see todos)
- Server-side auth middleware

---

## 1. 🗂️ Update Project Structure

We’ll need:
- A `User` model
- Auth-related server functions (`register`, `login`)
- Middleware to protect server functions
- Conditional rendering based on auth state

---

## 2. 🛠️ Setup Dependencies

Add these to root `Cargo.toml` under `[dependencies]`

```toml
jsonwebtoken = "8.3"
bcrypt = "0.11"
dotenvy = "0.15"
```

Also add to `my_todo_app_axum/Cargo.toml` if not already there.

---

## 3. 🧱 Create User Model & DB Table

### Define `User` struct in `crates/my_todo_app/src/lib.rs`

```rust
#[derive(Serialize, Deserialize, Clone, Debug)]
pub struct User {
    pub id: i32,
    pub email: String,
    pub password_hash: String,
}
```

### Add migration file: `migrations/0002_create_users.sql`

```sql
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL
);
```

Run the migration:

```bash
sqlx migrate run
```

---

## 4. 🔐 Auth Server Functions

In `lib.rs`, add:

```rust
use bcrypt::{hash, verify, DEFAULT_COST};
use jsonwebtoken::{encode, decode, Header, Validation, EncodingKey, DecodingKey};
use std::time::{SystemTime, UNIX_EPOCH};

#[derive(Deserialize, Serialize, Clone)]
struct Claims {
    sub: String,
    exp: usize,
}

const JWT_SECRET: &[u8] = b"your-secret-key-here";

#[server(Register)]
async fn register(email: String, password: String) -> Result<(), ServerFnError> {
    let pool = get_pool()?;
    let hash = hash(password, DEFAULT_COST).unwrap();

    sqlx::query!(
        "INSERT INTO users (email, password_hash) VALUES ($1, $2)",
        email,
        hash
    )
    .execute(&pool)
    .await?;

    Ok(())
}

#[server(Login)]
async fn login(email: String, password: String) -> Result<String, ServerFnError> {
    let pool = get_pool()?;
    let user = sqlx::query_as!(
        User,
        "SELECT * FROM users WHERE email = $1",
        email
    )
    .fetch_optional(&pool)
    .await?;

    match user {
        Some(user) => {
            let valid = verify(password, &user.password_hash).unwrap();
            if valid {
                let now = SystemTime::now().duration_since(UNIX_EPOCH).unwrap().as_secs() as usize;
                let claims = Claims {
                    sub: user.id.to_string(),
                    exp: now + 86400, // 24 hours
                };
                encode(&Header::default(), &claims, &EncodingKey::from_secret(JWT_SECRET))
                    .map_err(|e| ServerFnError::new(e.to_string()))
            } else {
                Err(ServerFnError::new("Invalid password"))
            }
        },
        None => Err(ServerFnError::new("User not found")),
    }
}
```

---

## 5. 🛡️ Auth Middleware (Axum)

Create a new file: `src/middleware/auth.rs` inside `my_todo_app_axum`

```rust
use axum::{
    async_trait,
    extract::{FromRequestParts, State},
    http::{request::Parts, StatusCode},
    RequestPartsExt,
};
use jsonwebtoken::{decode, DecodingKey, Validation, Algorithm};
use std::sync::Arc;
use crate::User;
use sqlx::PgPool;

const JWT_SECRET: &[u8] = b"your-secret-key-here";

#[derive(Clone)]
pub struct AuthUser {
    pub id: i32,
}

#[async_trait]
impl<S> FromRequestParts<S> for AuthUser
where
    S: Send + Sync,
{
    type Rejection = (StatusCode, String);

    async fn from_request_parts(parts: &mut Parts, state: &S) -> Result<Self, Self::Rejection> {
        let token = parts
            .headers
            .get("Authorization")
            .and_then(|h| h.to_str().ok())
            .and_then(|s| s.strip_prefix("Bearer "))
            .ok_or((StatusCode::UNAUTHORIZED, "Missing or invalid token".to_string()))?;

        let pool = parts.state::<Arc<PgPool>>()
            .ok_or((StatusCode::INTERNAL_SERVER_ERROR, "Database pool missing".to_string()))?;

        let decoded = decode::<serde_json::Value>(
            token,
            &DecodingKey::from_secret(JWT_SECRET),
            &Validation::new(Algorithm::HS256),
        ).map_err(|e| (StatusCode::UNAUTHORIZED, e.to_string()))?;

        let user_id: i32 = decoded.claims["sub"].as_str().unwrap().parse().unwrap();

        // Optional: Validate user exists
        let user = sqlx::query_as!(
            User,
            "SELECT * FROM users WHERE id = $1",
            user_id
        )
        .fetch_optional(pool.as_ref())
        .await
        .map_err(|e| (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()))?;

        if user.is_none() {
            return Err((StatusCode::UNAUTHORIZED, "User not found".to_string()));
        }

        Ok(AuthUser { id: user_id })
    }
}
```

Update `main.rs` to use it:

```rust
mod middleware;

use self::middleware::AuthUser;
use axum::extract::State;
use std::sync::Arc;

// Example protected route
async fn protected_route(
    _auth: AuthUser,
    State(pool): State<Arc<PgPool>>,
) -> Result<String, (StatusCode, String)> {
    Ok("You're authenticated!".to_string())
}
```

And add it to your router:

```rust
let app = Router::new()
    .route("/api/protected", get(protected_route))
    .leptos_routes(...)
```

---

## 6. 🖥️ Frontend: Login & Register Forms

Update `App()` in `lib.rs`:

```rust
#[component]
pub fn App() -> impl IntoView {
    let login_action = create_server_action::<Login>();
    let register_action = create_server_action::<Register>();

    view! {
        <div>
            <h2>"Register"</h2>
            <ActionForm action=register_action>
                <input type="text" name="email" placeholder="Email" />
                <input type="password" name="password" placeholder="Password" />
                <button type="submit">"Register"</button>
            </ActionForm>

            <h2>"Login"</h2>
            <ActionForm action=login_action>
                <input type="text" name="email" placeholder="Email" />
                <input type="password" name="password" placeholder="Password" />
                <button type="submit">"Login"</button>
            </ActionForm>
        </div>
    }
}
```

Now you can register and log in!

---

## ✅ Next:

1. Store the JWT in browser storage (localStorage) after login
2. Redirect to `/todos` only when logged in
3. Use Axios-like fetch wrappers for authenticated API calls
4. Deploy this app securely
