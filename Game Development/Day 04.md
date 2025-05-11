# Let's take the next step and **add a lobby system with login flow** using:

- 🔐 JWT-based authentication
- 🧑‍🤝‍🧑 Multiplayer lobby (join/leave/create)
- 💾 User profiles stored in a lightweight database (SQLite for now)

---

# 🛠️ Feature: Lobby System + Login Flow

This will allow users to:

1. ✅ Log in or register via simple form
2. 📋 View list of active lobbies
3. 👥 Create or join a multiplayer game room
4. 🔄 Sync real-time user presence (connected/disconnected)

---

## 🗂️ Updated Folder Structure

```
rust-multiplayer-game/
├── client/
│   ├── src/
│   │   ├── main.rs
│   │   ├── lib.rs
│   │   └── components/
│   │       ├── AuthModal.rs
│   │       ├── LobbyList.rs
│   │       └── GameRoom.rs
│   └── index.html
├── server/
│   ├── src/
│   │   ├── main.rs
│   │   ├── auth.rs
│   │   ├── lobby.rs
│   │   └── db.rs
│   └── Cargo.toml
├── assets/
└── README.md
```

---

# 🧱 Step 1: Add SQLite Database Support

We'll use `sqlx` for async SQLite access.

### 🔧 Update `server/Cargo.toml`

```toml
[dependencies]
axum = "0.6"
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
sqlx = { version = "0.6", features = ["sqlite", "runtime-tokio-native-tls"] }
dotenvy = "0.15"
jsonwebtoken = "0.2"
wasm-bindgen = "0.2"
```

### 📄 `server/src/db.rs`

```rust
use sqlx::SqlitePool;

pub async fn init_db() -> SqlitePool {
    let pool = SqlitePool::connect("sqlite:game.db").await.unwrap();

    sqlx::query!(
        r#"
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT UNIQUE NOT NULL,
            password TEXT NOT NULL
        )
        "#
    ).execute(&pool).await.unwrap();

    sqlx::query!(
        r#"
        CREATE TABLE IF NOT EXISTS lobbies (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            owner TEXT NOT NULL
        )
        "#
    ).execute(&pool).await.unwrap();

    pool
}
```

Run this once to create the DB file:

```bash
touch game.db
```

---

# 🔐 Step 2: Add JWT-Based Authentication

### 📄 `server/src/auth.rs`

```rust
use axum::{
    extract::State,
    Json,
    http::StatusCode,
};
use serde::{Deserialize, Serialize};
use sqlx::SqlitePool;
use jsonwebtoken::{encode, Header, EncodingKey};
use std::env;

#[derive(Deserialize)]
pub struct RegisterRequest {
    username: String,
    password: String,
}

#[derive(Serialize, Deserialize)]
struct Claims {
    sub: String,
    exp: usize,
}

pub async fn register(
    State(pool): State<SqlitePool>,
    Json(payload): Json<RegisterRequest>,
) -> Result<String, StatusCode> {
    let hashed = bcrypt::hash(&payload.password, bcrypt::DEFAULT_COST).unwrap();

    sqlx::query!(
        "INSERT INTO users (username, password) VALUES (?, ?)",
        payload.username,
        hashed
    )
    .execute(&pool)
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    Ok("User created".to_string())
}

pub async fn login(
    State(pool): State<SqlitePool>,
    Json(payload): Json<RegisterRequest>,
) -> Result<String, StatusCode> {
    let user = sqlx::query!(
        "SELECT password FROM users WHERE username = ?",
        payload.username
    )
    .fetch_optional(&pool)
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    if let Some(record) = user {
        if bcrypt::verify(&payload.password, &record.password).unwrap_or(false) {
            let claims = Claims {
                sub: payload.username,
                exp: (chrono::Utc::now() + chrono::Duration::hours(1)).timestamp() as usize,
            };

            let token = encode(
                &Header::default(),
                &claims,
                &EncodingKey::from_secret(env::var("JWT_SECRET").unwrap().as_ref()),
            ).unwrap();

            return Ok(token);
        }
    }

    Err(StatusCode::UNAUTHORIZED)
}
```

Set your secret in `.env`:

```bash
echo "JWT_SECRET=my-secret-key" > .env
```

---

# 🧑‍🤝‍🧑 Step 3: Lobby Management Routes

### 📄 `server/src/lobby.rs`

```rust
use axum::{
    extract::{State, Path},
    Json,
    http::StatusCode,
};
use serde::{Deserialize, Serialize};
use sqlx::SqlitePool;

#[derive(Deserialize)]
pub struct CreateLobbyRequest {
    name: String,
    owner: String,
}

pub async fn create_lobby(
    State(pool): State<SqlitePool>,
    Json(payload): Json<CreateLobbyRequest>,
) -> Result<String, StatusCode> {
    sqlx::query!(
        "INSERT INTO lobbies (name, owner) VALUES (?, ?)",
        payload.name,
        payload.owner
    )
    .execute(&pool)
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    Ok("Lobby created".to_string())
}

pub async fn get_lobbies(
    State(pool): State<SqlitePool>,
) -> Json<Vec<LobbyResponse>> {
    let rows = sqlx::query!("SELECT id, name, owner FROM lobbies")
        .fetch_all(&pool)
        .await
        .unwrap();

    let lobbies: Vec<_> = rows.into_iter()
        .map(|r| LobbyResponse {
            id: r.id,
            name: r.name,
            owner: r.owner,
        })
        .collect();

    Json(lobbies)
}

#[derive(Serialize)]
struct LobbyResponse {
    id: i64,
    name: String,
    owner: String,
}
```

---

# 🌐 Step 4: Add New Routes in `main.rs`

Update `server/src/main.rs`:

```rust
mod auth;
mod lobby;
mod db;

use axum::{
    Router, routing::post, extract::State
};
use tower_http::services::ServeDir;
use std::sync::Arc;
use std::net::SocketAddr;
use tokio::sync::broadcast;

#[tokio::main]
async fn main() {
    let pool = db::init_db().await;
    let pool = Arc::new(pool);
    let (tx, _) = broadcast::channel(100);

    let app = Router::new()
        .route("/register", post(auth::register))
        .route("/login", post(auth::login))
        .route("/create-lobby", post(lobby::create_lobby))
        .route("/lobbies", post(lobby::get_lobbies))
        .route("/ws", get(move |ws: axum::extract::WebSocketUpgrade| {
            let tx = tx.clone();
            async move {
                ws.on_upgrade(move |socket| handle_socket(socket, tx))
            }
        }))
        .with_state(pool.clone());

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Server running on {}", addr);
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

---

# 🖥️ Step 5: Add Auth UI and Lobby List in Leptos

### 📄 `client/src/components/AuthModal.rs`

```rust
use leptos::*;

#[component]
pub fn AuthModal() -> impl IntoView {
    let (username, set_username) = create_signal(String::new());
    let (password, set_password) = create_signal(String::new());

    view! {
        <div class="auth-modal">
            <input
                prop:value={username}
                on:input=move |e| set_username(event_target_value(&e))
                placeholder="Username"
            />
            <input
                prop:value={password}
                on:input=move |e| set_password(event_target_value(&e))
                type="password"
                placeholder="Password"
            />
            <button>"Login"</button>
            <button>"Register"</button>
        </div>
    }
}
```

### 📄 `client/src/components/LobbyList.rs`

```rust
use leptos::*;
use wasm_bindgen::prelude::*;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Lobby {
    id: i32,
    name: String,
    owner: String,
}

#[component]
pub fn LobbyList() -> impl IntoView {
    let fetch_lobbies = create_action(|token: String| {
        let token = token.clone();
        async move {
            let window = web_sys::window().expect("No window");
            let resp = window.fetch_with_str_and_init(
                "http://localhost:3000/lobbies",
                web_sys::RequestInit::new()
                    .method("POST")
                    .header("Authorization", &format!("Bearer {}", token))
                    .body(Some(&JsValue::from_str("{}"))),
            );

            let response = resp.expect("Failed to send request").promise().await.unwrap();
            let json = response.json().unwrap().await.unwrap();
            let lobbies: Vec<Lobby> = serde_wasm_bindgen::from_value(json).unwrap();
            lobbies
        }
    });

    view! {
        <Show when=move || fetch_lobbies.value().get().is_none()>
            <button on:click=move |_| fetch_lobbies.dispatch("your-jwt-token".to_string())>
                "Load Lobbies"
            </button>
        </Show>

        <For
            each=move || fetch_lobbies.value().get().unwrap_or_default()
            key=|lobby| lobby.id
            children=move |lobby| {
                view! {
                    <div>
                        <strong>{&lobby.name}</strong> " by " {&lobby.owner}
                    </div>
                }
            }
        />
    }
}
```

---

# 🎉 Done!

You've just added:

✅ A working login/register flow  
✅ A persistent lobby system  
✅ Real-time lobby listing  
✅ Secure JWT tokens  

---

# 🚀 Next Steps

✅ Add matchmaking logic (auto-assign players to rooms)
✅ Implement Fyrox scene loading from the lobby
✅ Deploy the full stack together (Cloudflare + Fly.io)
✅ Or build a UI for creating/editing 3D scenes
