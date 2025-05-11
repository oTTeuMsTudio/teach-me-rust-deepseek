# Full Stack Web Development with a 3D Multiplayer Game in Rust using Leptos

- **Frontend**: Rust + [Leptos](https://leptos.dev/) for reactive web UI
- **3D Rendering**: Use `wasm-bindgen`, `web-sys`, and `three-rs` or `glam`/`wgpu`
- **Multiplayer Logic**: WebSockets (or similar) for real-time communication
- **Backend**: Rust with Axum or Actix Web for the server

Let’s break it down into **modules**, and by the end, you’ll have a working **3D multiplayer game** built entirely in Rust!

---

## 🧠 Prerequisites

Before we begin, ensure you know:
- Basics of **Rust**
- Basic **HTML/CSS/JS concepts**
- Familiarity with **WebGL / 3D graphics concepts** (optional)
- Git & terminal basics

---

# 🔥 Full Stack Web Dev Course with Rust & Leptos – Build a 3D Multiplayer Game

---

## 📦 Module 0: Setting Up Your Environment

### Tools to Install

1. **Rust Toolchain**
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

2. **wasm-pack**
   ```bash
   cargo install wasm-pack
   ```

3. **Trunk** (build tool for Leptos apps)
   ```bash
   cargo install trunk
   ```

4. **Node.js** (for bundling dev tools if needed)

5. **VSCode + Rust Analyzer**

---

## 🧱 Module 1: Intro to Leptos (Frontend Framework in Rust)

We will use **Leptos** to build the frontend.

### Create a New Leptos App

```bash
cargo leptos new my_3d_game
cd my_3d_game
```

This sets up a basic app structure.

### Understanding Leptos Concepts

- **Reactive Programming Model**
- Components (`#[component]`)
- Signals (`create_signal`, `create_rw_signal`)
- Effects (`create_effect`)
- Server Functions (`#[server]`)

### Example Counter App

```rust
use leptos::*;

#[component]
fn App() -> impl IntoView {
    let (count, set_count) = create_signal(0);

    view! {
        <button on:click=move |_| set_count.update(|n| *n += 1)>
            "Click me: " {count}
        </button>
    }
}

fn main() {
    mount_to_body(App);
}
```

Run with:
```bash
trunk serve
```

Visit `http://localhost:8080`

---

## 🎮 Module 2: Add 3D Graphics Using WebGL/WGPU in Rust

We’ll render a simple 3D scene using Rust bindings.

### Option A: Use `three-rs` (Rust bindings for Three.js)

Three.js is the most beginner-friendly 3D engine for the web.

1. Add to `Cargo.toml`:

```toml
[dependencies]
three-rs = "0.6"
wasm-bindgen = "0.2"
web-sys = { version = "0.3", features = ["WebGlRenderingContext"] }
```

2. Initialize a simple 3D Scene in your Leptos component:

```rust
use leptos::*;
use three_rs::prelude::*;

#[component]
fn ThreeScene() -> impl IntoView {
    // This runs on the browser (wasm)
    spawn_local(async move {
        let mut app = App::builder()
            .with_window_size(800, 600)
            .build()
            .unwrap();

        let camera = PerspectiveCamera::new(75.0, 800.0 / 600.0, 0.1, 1000.0);
        let renderer = WebGLRenderer::new(&app.gl);
        renderer.set_size(800.0, 600.0, false);
        document().body().unwrap().append_child(&renderer.dom_element()).unwrap();

        let geometry = BoxGeometry::new(1.0, 1.0, 1.0);
        let material = MeshBasicMaterial::new(Color::new(0.0, 1.0, 0.0));
        let cube = Mesh::new(geometry, material);
        app.scene.add(&cube);
        app.camera = Some(Box::new(camera));

        loop {
            cube.rotation_mut().y += 0.01;
            cube.rotation_mut().x += 0.01;

            renderer.render(&app.scene, &*app.camera.as_ref().unwrap());
            app.frame().await;
        }
    });

    view! {}
}

// In App component:
view! {
    <ThreeScene />
}
```

> Note: You may need to adjust imports and setup based on current `three-rs` API.

---

## ⚙️ Module 3: Backend with Rust (Axum or Actix)

We’ll use **Axum** for our backend.

### Setup Axum Server

Create a new binary in the same workspace:

```bash
cargo new backend --bin
```

Add to `backend/Cargo.toml`:

```toml
[dependencies]
axum = "0.6"
tokio = { version = "1", features = ["full"] }
tower-http = "0.3"
serde = { version = "1.0", features = ["derive"] }
```

### Simple WebSocket Server for Multiplayer

Use `tungstenite` or `axum-websockets`.

Example WebSocket Echo Server:

```rust
use axum::{
    extract::ws::{WebSocket, WebSocketUpgrade},
    response::IntoResponse,
};
use std::net::SocketAddr;

async fn ws_handler(ws: WebSocketUpgrade) -> impl IntoResponse {
    ws.on_upgrade(|socket| async {
        handle_socket(socket).await;
    })
}

async fn handle_socket(mut socket: WebSocket) {
    while let Some(msg) = socket.recv().await {
        match msg {
            Ok(text) => {
                println!("Received: {:?}", text);
                socket.send(text).await.unwrap();
            },
            Err(e) => {
                println!("Error: {}", e);
                break;
            }
        }
    }
}

#[tokio::main]
async fn main() {
    let app = axum::Router::new()
        .route("/ws", axum::routing::get(ws_handler));

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Listening at http://{}", addr);
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

Run with:

```bash
cargo run --bin backend
```

---

## 🌐 Module 4: Connect Frontend to Backend (WebSockets)

In your Leptos frontend:

```rust
use wasm_bindgen::JsCast;
use web_sys::WebSocket;

let ws = WebSocket::new("ws://localhost:3000/ws").unwrap();
let ws = ws.dyn_into::<web_sys::WebSocket>().unwrap();

let onmessage = Closure::wrap(Box::new(move |e: MessageEvent| {
    if let Ok(ab) = e.data().dyn_into::<js_sys::ArrayBuffer>() {
        let data = js_sys::Uint8Array::new(&ab).to_vec();
        log::info!("Received bytes: {:?}", data);
    }
}) as Box<dyn FnMut(MessageEvent)>);

ws.set_onmessage(Some(onmessage.as_ref().unchecked_ref()));
```

You can now send/receive messages from the client.

---

## 🕹️ Module 5: Implementing Multiplayer Logic

Each player sends their position/state via WebSocket.

### Player Struct

```rust
#[derive(Serialize, Deserialize)]
struct PlayerUpdate {
    id: String,
    x: f32,
    y: f32,
    z: f32,
}
```

On the server, maintain a list of connected players.

On the client:
- Send position updates every frame
- Listen for other players' positions and update the scene accordingly

---

## 🧬 Module 6: Final Polish and Deployment

### Features to Add:
- Player avatars (simple cubes or models)
- Collision detection
- Scoreboard
- Chat (bonus)

### Deploying:
- Frontend: Host via GitHub Pages, Vercel, Netlify (via Trunk)
- Backend: Docker container or deploy to AWS/Azure/Render/etc.

---

## 📁 Project Structure Overview

```
my-multiplayer-game/
├── Cargo.toml
├── frontend/
│   ├── Cargo.toml
│   └── src/main.rs
├── backend/
│   ├── Cargo.toml
│   └── src/main.rs
└── README.md
```

---

## 🧪 Bonus Challenges

- Add physics with `rapier3d`
- Add model loading (`.glb`, `.gltf`)
- Add matchmaking logic
- Add auth system (JWT)

---

## 📚 Resources

- [Leptos Docs](https://leptos.dev/)
- [three-rs GitHub](https://github.com/asny/three-rs)
- [Axum Docs](https://docs.rs/axum/latest/axum/)
- [wasm-bindgen Guide](https://rustwasm.github.io/docs/wasm-bindgen/)
- [WebSockets in Rust](https://github.com/snapview/tide-websockets)

---

## ✅ Final Output:

✅ A **3D cube rotating** game  
✅ Real-time **multiplayer sync** over WebSockets  
✅ A **Leptos frontend** in Rust  
✅ A **Rust backend** with Axum  
✅ Ability to expand into full games or interactive 3D web apps
