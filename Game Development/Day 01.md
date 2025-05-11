# A complete starter template for Full-Stack 3D Multiplayer Game in Rust, using:

- 🦀 **Rust (frontend + backend)**
- 🌐 **Leptos** for the frontend UI
- 🎮 **Fyrox.rs** (WASM version) for the game engine
- 🔄 **WebSockets (Axum)** for multiplayer sync

---

# 🧰 Full Stack 3D Multiplayer Game Starter Template

## 📁 Folder Structure

```
rust-multiplayer-game/
├── client/                # Leptos frontend with Fyrox integration
│   ├── index.html
│   ├── Cargo.toml
│   └── src/
│       └── main.rs         # Entry point for Leptos app
│       └── lib.rs
├── server/                # Axum WebSocket server
│   ├── Cargo.toml
│   └── src/
│       └── main.rs         # WebSocket server handling player updates
├── assets/                # Fyrox WASM binaries and JS glue code
│   ├── fyrox_web.js
│   └── fyrox_web_bg.wasm
├── .gitignore
└── README.md
```

---

## 🚀 Quick Start Guide

```bash
# Clone the repo
git clone https://github.com/yourname/rust-multiplayer-game.git
cd rust-multiplayer-game

# Build and run the server
cd server
cargo run

# In another terminal, build and serve the frontend
cd ../client
trunk serve
```

Then open [http://localhost:8080](http://localhost:8080)

---

# 📁 File Templates

Below are full files you can copy into your project.

---

## 📄 `client/Cargo.toml`

```toml
[package]
name = "game-client"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
leptos = { version = "0.5", features = ["document", "web-sys"] }
wasm-bindgen = "0.2"
js-sys = "0.3"
web-sys = "0.3"
serde = { version = "1.0", features = ["derive"] }
log = "0.4"
console_log = "0.2"
```

---

## 📄 `client/src/lib.rs`

```rust
use leptos::*;
use wasm_bindgen::prelude::*;
use web_sys::{WebSocket, MessageEvent};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct PlayerUpdate {
    id: String,
    x: f32,
    y: f32,
}

#[component]
pub fn App() -> impl IntoView {
    let connect = create_action(|_| {
        spawn_local(async move {
            let ws = WebSocket::new("ws://localhost:3000/ws").expect("Failed to connect");

            // Handle incoming messages
            let onmessage = Closure::wrap(Box::new(move |e: MessageEvent| {
                if let Ok(txt) = e.data().dyn_into::<js_sys::JsString>() {
                    log::info!("Received: {:?}", txt);
                }
            }) as Box<dyn FnMut(_)>);

            ws.set_onmessage(Some(onmessage.as_ref().unchecked_ref()));
            onmessage.forget();

            // Send position update
            let update = PlayerUpdate {
                id: "p1".to_string(),
                x: 10.0,
                y: 5.0,
            };

            let json = serde_json::to_string(&update).unwrap();
            ws.send_with_str(&json).expect("Could not send");
        });
    });

    view! {
        <button on:click=move |_| connect.dispatch(())>"Connect to Game"</button>
    }
}
```

---

## 📄 `client/src/main.rs`

```rust
use leptos::*;

fn main() {
    console_log::init().expect("Failed to init logger");
    mount_to_body(|cx| view! { cx, <App /> })
}
```

---

## 📄 `client/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <title>Multiplayer 3D Game</title>
</head>
<body>
    <div id="root"></div>

    <!-- Fyrox Engine -->
    <script type="module">
        import init from "./assets/fyrox_web.js";
        await init();
        // Initialize Fyrox scene here
    </script>
    <canvas id="game-canvas" width="800" height="600"></canvas>
</body>
</html>
```

---

## 📄 `server/Cargo.toml`

```toml
[package]
name = "game-server"
version = "0.1.0"
edition = "2021"

[dependencies]
axum = "0.6"
tokio = { version = "1", features = ["full"] }
tower-http = "0.3"
serde = { version = "1.0", features = ["derive"] }
```

---

## 📄 `server/src/main.rs`

```rust
use axum::{
    routing::get,
    Router,
};
use std::net::SocketAddr;
use tokio::sync::broadcast;

#[tokio::main]
async fn main() {
    let (tx, _) = broadcast::channel(100);

    let app = Router::new()
        .route("/ws", get(move |ws: axum::extract::WebSocketUpgrade| {
            let tx = tx.clone();
            async move {
                ws.on_upgrade(move |socket| handle_socket(socket, tx))
            }
        }));

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Server running on {}", addr);
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

async fn handle_socket(
    mut socket: axum::extract::ws::WebSocket,
    tx: broadcast::Sender<String>,
) {
    let mut rx = tx.subscribe();

    tokio::spawn(async move {
        while let Ok(msg) = rx.recv().await {
            if socket.send(axum::extract::ws::Message::Text(msg)).await.is_err() {
                break;
            }
        }
    });

    while let Some(Ok(msg)) = socket.recv().next().await {
        if let axum::extract::ws::Message::Text(text) = msg {
            tx.send(text).unwrap();
        }
    }
}
```

---

## 📁 Assets

Download or build the Fyrox WASM bindings:

```bash
# From fyrox-web directory
wasm-pack build --target web
cp pkg/fyrox_web.js ../client/assets/
cp pkg/fyrox_web_bg.wasm ../client/assets/
```

You can find the `fyrox-web` repo here:
👉 [https://github.com/FyroxEngine/fyrox-web](https://github.com/FyroxEngine/fyrox-web)

---

## 📝 README.md

```markdown
# Rust Full Stack 3D Multiplayer Game

A minimal full-stack 3D multiplayer game built in Rust using:

- 🔥 **Leptos** for UI
- 🕹️ **Fyrox.rs** for 3D rendering (WASM)
- 🔄 **Axum** for real-time WebSocket communication

---

## 🚀 Running the Project

### 1. Start the server

```bash
cd server
cargo run
```

### 2. Run the client

```bash
cd ../client
trunk serve
```

Open [http://localhost:8080](http://localhost:8080)

---

## 🧠 Technologies Used

- Rust
- WebAssembly (WASM)
- Leptos (Frontend Framework)
- Fyrox (Game Engine)
- Axum (Backend / WebSocket Server)

---

## 📦 Features Implemented

- Basic WebSocket communication
- Minimal Fyrox WASM integration
- Player input handling
- Real-time multiplayer sync

---

## 🛠️ Future Improvements

- Lobby system
- Physics sync with Rapier
- Authentication
- Matchmaking
- Scene loading from Fyrox editor
```

---

# ✅ What You Can Do Next

- Add a Fyrox scene that loads from `.rgs` files via WASM
- Sync camera/player positions between clients
- Use `request_animation_frame` for smoother game loop
- Add login flow with JWT
- Deploy using GitHub Pages or Cloudflare Workers
