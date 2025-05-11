# Full-stack web application with a 3D multiplayer game using Rust

We’re aiming to build:

- A **frontend** in **Rust using Leptos**
- A **3D game engine** using **Fyrox.rs**
- A **multiplayer experience**, likely with WebSockets
- And possibly a **backend** (also in Rust) handling game state synchronization

Let’s go through process step-by-step.

---

## 🎯 Project Overview

### 🧩 Stack Summary

| Layer       | Tech Used                          |
|------------|------------------------------------|
| Language    | Rust                               |
| Frontend    | [Leptos](https://leptos.dev/)      |
| Game Engine | [Fyrox.rs](https://fyroxx.re/)     |
| Network     | WebSocket (possibly with Axum or Warp) |
| Build Tool  | Trunk / wasm-bindgen                |

---

# 🚀 Step-by-Step Full Stack Guide

---

## 🧱 1. Set Up Your Development Environment

Install tools:

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Add WASM target
rustup target add wasm32-unknown-unknown

# Install trunk (build tool for Leptos)
cargo install trunk

# Install cargo-generate (for templates)
cargo install cargo-generate
```

---

## 🖥️ 2. Create a Leptos Frontend App

Generate a new Leptos app:

```bash
cargo generate --git https://github.com/leptos-rs/start-parcel.git --name my-game-ui
cd my-game-ui
```

Then run it:

```bash
trunk serve
```

We’ll see a basic counter app in our browser at `http://localhost:8080`.

---

## 🕹️ 3. Integrate Fyrox into Your Leptos App

> ⚠️ Fyrox currently doesn't support WebAssembly directly out of the box, but there are ongoing efforts. You can either:
>
> - Use the official `fyrox-web` branch (experimental)
> - Or embed a `<canvas>` and load Fyrox via WebAssembly manually.

We'll go the easiest route: embedding Fyrox as a WASM module inside the Leptos app.

### Option: Use `fyrox-web` (Experimental)

#### Clone fyrox-web:

```bash
git clone https://github.com/FyroxEngine/fyrox-web
cd fyrox-web
```

Build it:

```bash
wasm-pack build --target web
```

Now import it into your Leptos project:

```bash
cp pkg/fyrox_web_bg.wasm ../my-game-ui/assets/
cp pkg/fyrox_web.js ../my-game-ui/assets/
```

In our `index.html`:

```html
<script type="module">
  import init from "./assets/fyrox_web.js";
  await init();
  // Start the engine here
</script>
<canvas id="game-canvas"></canvas>
```

We can now control the canvas from JavaScript/Rust interop.

💡 For full integration, use `wasm-bindgen` to call Rust functions from Leptos components.

---

## 🌐 4. Set Up Multiplayer Backend

Use **Axum** + **Tokio** + **WebSocket** for real-time communication.

Create a new binary crate for the backend:

```bash
cargo new game-server --bin
cd game-server
```

Add dependencies in `Cargo.toml`:

```toml
[dependencies]
axum = "0.6"
tokio = { version = "1", features = ["full"] }
tower-http = "0.3"
serde = { version = "1.0", features = ["derive"] }
```

Example WebSocket server (`main.rs`):

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

Run the server:

```bash
cargo run
```

---

## 🤝 5. Connect Frontend to Backend

In your Leptos app, connect to the WebSocket server:

```rust
use leptos::*;
use wasm_bindgen::prelude::*;
use wasm_bindgen::JsCast;
use web_sys::WebSocket;

#[component]
fn App() -> impl IntoView {
    let connect = create_action(|_| {
        let ws = WebSocket::new("ws://localhost:3000/ws").expect("Failed to connect");

        ws.set_onmessage(Some({
            Closure::<dyn FnMut(_)>::new(move |e: MessageEvent| {
                if let Ok(txt) = e.data().dyn_into::<js_sys::JsString>() {
                    log::info!("Received: {:?}", txt);
                }
            })
            .into_js_value()
        }));

        move |_| {}
    });

    view! {
        <button on:click=move |_| connect.dispatch(())>"Connect"</button>
    }
}
```

---

## 🎮 6. Add Basic Multiplayer Movement

Each player sends their position to the server, which broadcasts it to others.

### On Client:

```rust
// Inside component
let send_position = move |x: f32, y: f32| {
    if let Ok(mut ws) = ws.lock() {
        let pos = format!("{{\"id\": \"p1\", \"x\": {}, \"y\": {}}}", x, y);
        ws.send_with_str(&pos).expect("Failed to send");
    }
};
```

Call this when moving the character in Fyrox.

### On Server:

Broadcast incoming messages to all clients.

Update `handle_socket()` to:

```rust
tx.send(text).unwrap(); // Broadcast
```

On client, update positions of other players accordingly.

---

## 📦 7. Package Everything Together

Use `Trunk` to bundle everything (Leptos + Fyrox WASM + JS glue).

Make sure assets like `.wasm`, `.js`, etc., are placed under `/assets`.

---

## 🧪 8. Testing & Debugging

- Use browser DevTools to inspect WebSocket traffic.
- Use `log` crate + `console_log` to debug Rust code in browser.
- Use Fyrox editor for scene creation (if available).

---

## 🧠 Tips & Gotchas

- **Fyrox in WASM is experimental**: It may not support all features yet.
- **Leptos + WASM**: Works well for UI, but avoid heavy computation.
- **Interop**: Use `wasm-bindgen` to pass data between Leptos and Fyrox.
- **State Syncing**: Consider authoritative server model to prevent cheating.
- **Performance**: Optimize rendering loops and network updates.

---

## 🧩 Optional Enhancements

- Use `bevy` instead of Fyrox (but no WASM support yet).
- Add physics sync with `rapier3d`.
- Add login system using JWT.
- Store game data in a database (e.g., SQLite, PostgreSQL via `sqlx`).
- Deploy using GitHub Pages or Cloudflare Workers.

---

## 🛠️ Example Folder Structure

```
project-root/
├── my-game-ui/         # Leptos frontend
│   ├── src/
│   └── assets/         # Fyrox WASM, JS glue
├── game-server/        # Axum backend
└── fyrox-web/          # Fyrox WASM build (optional)
```

---

## 🧭 Next Steps

1. ✅ Build a minimal Fyrox scene in WASM
2. ✅ Get input controls working
3. ✅ Send player input over WebSocket
4. ✅ Render other players’ movements
5. ✅ Add matchmaking / lobby system
6. ✅ Deploy to production

---

## 📘 Resources

- [Leptos Docs](https://leptos.dev/)
- [Fyrox Documentation](https://fyroxx.re/)
- [WASM Book](https://rustwasm.github.io/docs/book/)
- [Axum Examples](https://github.com/tokio-rs/axum/tree/main/examples)
