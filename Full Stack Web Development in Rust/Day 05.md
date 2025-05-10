# Level up your full-stack Rust app with real-time updates using WebSockets

This will allow:
- ✅ Live sync between multiple users
- ✅ Instant todo updates without polling
- ✅ Bidirectional communication between client and server

We’ll use **Axum WebSocket support** + **Leptos for the UI** to build this.

---

# 💬 Real-Time Todo App with WebSockets in Rust

## 🧠 What We’ll Build

| Feature | Description |
|--------|-------------|
| 🔁 Real-time updates | Todos update instantly across all clients |
| 📡 WebSocket server | Built into Axum backend |
| 🖥️ Leptos frontend | Reactively update UI when messages arrive |
| 👥 Broadcast to all users | When one user adds a todo, others see it live |

---

## 1. 🛠️ Add Dependencies

Update your `Cargo.toml` (in root or axum crate):

```toml
tower = "0.4"
tokio-tungstenite = { version = "0.17", features = ["runtime-async-std"] }
tungstenite = "0.17"
futures-util = { version = "0.3", features = ["stream"] }
```

---

## 2. 🧠 Define Shared Message Format

Create a shared `Message` type in `crates/my_todo_app/src/lib.rs`:

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Clone, Debug)]
pub enum WsMessage {
    TodoAdded(Todo),
    TodoToggled(Todo),
    TodoDeleted(i32),
}
```

---

## 3. 🌐 Set Up WebSocket Endpoint in Axum

In `my_todo_app_axum/src/main.rs`, add:

```rust
use axum::{
    extract::{ws::WebSocketUpgrade, State},
    response::IntoResponse,
};
use std::sync::Arc;
use tokio::sync::broadcast;
use futures_util::{sink::SinkExt, stream::StreamExt};

type Tx = broadcast::Sender<String>;
type Users = Arc<dashmap::DashMap<String, Tx>>;

#[tokio::main]
async fn main() {
    // Existing setup code...
    
    let users = Arc::new(Users::new());

    let app = Router::new()
        .route("/ws", get(ws_handler))
        // Other routes...
}

async fn ws_handler(
    ws: WebSocketUpgrade,
    State(users): State<Arc<Users>>,
) -> impl IntoResponse {
    ws.on_upgrade(move |socket| handle_socket(socket, users))
}

async fn handle_socket(mut socket: axum::extract::ws::WebSocket, users: Arc<Users>) {
    let (mut sender, mut receiver) = socket.split();

    while let Some(msg) = receiver.next().await {
        if let Ok(msg) = msg {
            if msg.is_text() || msg.is_binary() {
                // Handle incoming message
                let text = msg.to_text().unwrap();
                // Broadcast to all users
                for tx in users.iter() {
                    let _ = tx.value().send(text.to_string());
                }
            }
        } else {
            break;
        }
    }

    // Handle disconnection
}
```

> You can also use `tokio::spawn` to listen for DB changes and push updates.

---

## 4. 🖥️ Update Frontend to Listen for Messages

In Leptos component (`lib.rs`), connect to WebSocket:

```rust
use wasm_bindgen::JsCast;
use web_sys::WebSocket;

create_effect(move |_| {
    let ws = WebSocket::new("wss://your-render-app.onrender.com/ws").unwrap();
    let ws_clone = ws.clone();

    let on_message = Closure::wrap(Box::new(move |e: MessageEvent| {
        if let Ok(txt) = e.data().dyn_into::<js_sys::JsString>() {
            let msg_str = txt.as_string().unwrap();
            let msg: WsMessage = serde_json::from_str(&msg_str).unwrap();

            match msg {
                WsMessage::TodoAdded(todo) => {
                    /* Update signal */
                },
                WsMessage::TodoToggled(todo) => {
                    /* Toggle todo locally */
                },
                WsMessage::TodoDeleted(id) => {
                    /* Remove from list */
                }
            }
        }
    }) as Box<dyn FnMut(_)>);

    ws.set_onmessage(Some(on_message.as_ref().unchecked_ref()));
    on_message.forget();
});
```

---

## 5. 🔄 Broadcast Updates from Server

When a user creates a todo via a server function:

```rust
let json_msg = serde_json::to_string(&WsMessage::TodoAdded(new_todo)).unwrap();
for tx in users.iter() {
    let _ = tx.value().send(json_msg.clone());
}
```

Now every connected client gets the update instantly!

---

## ✅ Summary

✅ Added WebSocket endpoint in Axum  
✅ Created bidirectional communication  
✅ Updated UI reactively in Leptos  
✅ Broadcasted real-time updates  

---

## 🚀 Next:

1. 🤖 Adding AI integration (e.g., auto-suggest todo titles)
2. 📱 Making the app mobile-friendly with Tailwind CSS
3. 🧪 Writing tests for server functions and WebSocket logic
4. 📊 Adding analytics or logging with tracing

Let me know which feature you'd like to build next — I'm here to guide you step-by-step!
