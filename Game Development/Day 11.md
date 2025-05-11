# build momentum” phase

# Time to plan our next feature or game built on top of our Rust multiplayer engine.

---

# 🚀 Vision: Your Next Step

We’ll build a **simple but fun multiplayer 3D arena shooter**, using your full-stack Rust engine, with:

- 🎮 WASM-powered Fyrox rendering
- 🧍 Player movement sync
- 🔫 Shooting mechanics
- 🧠 AI bots (optional)
- 🏆 Scoreboard + leaderboard
- 💥 Real-time physics & collisions

Let’s call it:

## 🔫 "Rust Arena"

> A fast-paced, browser-based 3D shooter for 2–8 players.

---

# 📋 MVP Feature List

| Feature | Description |
|--------|-------------|
| 🧍 Player Movement | WASD + mouse look (Fyrox camera) |
| 🔫 Weapon System | Basic shooting (raycasting) |
| 💥 Damage System | Health bars, death events |
| 🧠 AI Bots | Optional basic pathfinding bots |
| 🧑‍🤝‍🧑 Lobby System | Join/leave rooms |
| 🏁 Match Start/End | Countdown, win conditions |
| 📊 Leaderboard | Redis-backed score tracking |
| 🗺️ Level Editor | In-browser level building (from earlier) |

---

# 🎮 Game Design Overview

### 🧩 Core Loop

1. Players join a lobby.
2. Game starts when 2+ players are ready.
3. Players move around an arena, shoot each other.
4. First to reach 10 kills wins.
5. Scores are saved to leaderboard.

---

# 🧱 Folder Structure for New Game

```
rust-multiplayer-game/
├── client/
│   └── src/
│       ├── components/
│       │   ├── LobbyList.rs
│       │   ├── GameRoom.rs
│       │   ├── PlayerHUD.rs
│       │   └── LevelEditor.rs
│       └── systems/
│           ├── input.rs
│           ├── network.rs
│           └── physics.rs
├── server/
│   └── src/
│       ├── systems/
│       │   ├── matchmaker.rs
│       │   ├── combat.rs
│       │   └── sync.rs
│       └── entities/
│           ├── player.rs
│           └── projectile.rs
└── assets/
    ├── scenes/
    │   └── arena.rgs
    └── sounds/
        ├── shoot.mp3
        └── hit.mp3
```

---

# 🕹️ Step-by-Step Development Plan

Let’s break this into phases you can ship incrementally.

---

## 🧪 Phase 1: Setup & Scene Load

✅ Goal: Load a Fyrox scene and show one player

### Tasks:
- Load `arena.rgs` from `/assets/scenes`
- Set up a first-person camera
- Display debug logs for FPS, position
- Add WASD movement (local only)

### Tools:
- Fyrox WASM bindings
- Leptos DOM integration
- Trunk + wasm-bindgen

---

## 🧍 Phase 2: Local Player Controls

✅ Goal: Move player locally and display in scene

### Tasks:
- Implement keyboard/mouse input
- Update Fyrox camera/player position
- Show local player mesh
- Add crosshair UI

### Code Snippet (`input.rs`):

```rust
use leptos::*;
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
}

pub fn setup_input() {
    let window = web_sys::window().expect("no global window exists");
    let closure = Closure::wrap(Box::new(move |event: web_sys::KeyboardEvent| {
        match event.key().as_str() {
            "w" => move_forward(),
            "s" => move_backward(),
            "a" => strafe_left(),
            "d" => strafe_right(),
            _ => {}
        }
    }) as Box<dyn FnMut(_)>);

    window.add_event_listener_with_callback("keydown", closure.as_ref().unchecked_ref())
        .unwrap();
    closure.forget();
}

fn move_forward() { /* TODO: update player transform */ }
```

---

## 🌐 Phase 3: Sync Players Over WebSocket

✅ Goal: See other players in the game

### Tasks:
- Send player position every frame
- Broadcast positions via Axum WS
- Render remote players as clones
- Smooth interpolation between positions

### Server Side (`sync.rs`):

```rust
use axum::extract::ws::{WebSocket, Message};
use std::sync::broadcast;
use serde_json::Value;

pub async fn handle_socket(mut socket: WebSocket, tx: broadcast::Sender<String>) {
    let mut rx = tx.subscribe();

    tokio::spawn(async move {
        while let Ok(msg) = rx.recv().await {
            if socket.send(Message::Text(msg)).await.is_err() {
                break;
            }
        }
    });

    while let Some(Ok(Message::Text(text))) = socket.recv().next().await {
        // Broadcast to all players
        tx.send(text.clone()).unwrap();
    }
}
```

---

## 🔫 Phase 4: Add Shooting Mechanics

✅ Goal: Shoot projectiles and detect hits

### Tasks:
- Add raycasting on mouse click
- Spawn visual projectile
- Detect collision with other players
- Apply damage and send kill message

### Code Snippet (`combat.rs`):

```rust
pub fn check_hit(ray_origin: Vec3, ray_dir: Vec3) -> Option<PlayerId> {
    // Iterate over players and check intersection
    for player in &players {
        if ray_intersects_sphere(ray_origin, ray_dir, player.position, 1.0) {
            return Some(player.id);
        }
    }
    None
}
```

---

## 🏆 Phase 5: Add Leaderboard & Win Conditions

✅ Goal: Track scores and end matches

### Tasks:
- Track kills/deaths per player
- Save to Redis leaderboard
- End match at 10 kills
- Display winner screen

---

## 🧠 Bonus: Add AI Bots (Optional)

✅ Goal: Play against bots when not enough players

### Tasks:
- Use simple pathfinding or patrol routes
- Fire randomly or target nearest player
- Use same physics/sync system

---

# 🧪 Testing Strategy

| Type | Tool |
|------|------|
| Local Dev | `trunk serve`, `cargo run` |
| Unit Tests | `#[cfg(test)]` |
| E2E Tests | Puppeteer + WebSocket mock |
| Multiplayer Test | Open multiple tabs |

---

# 🚀 Deployment Checklist

| Item | Status |
|------|--------|
| Deploy Frontend | ✅ Done |
| Deploy Backend | ✅ Done |
| Upload Assets | ✅ Done |
| Set Up Lobby API | ✅ Done |
| Enable Redis | ✅ Done |
| Create Demo Video | 🔄 In Progress |
| Launch Landing Page | ✅ Done |
| Submit to Communities | 🔄 In Progress |

---

# 🎯 Final Vision

You’re not just building a game — you’re building a **platform** that others can extend.

With this plan, you'll be able to:

✅ Launch a working multiplayer demo  
✅ Showcase Rust's power for game dev  
✅ Inspire others to try Rust full-stack  
✅ Turn this into a portfolio piece or open-source project  

---

# 🚨 Next:

✅ Generate a **GitHub Project Board** for this roadmap
✅ **write a README for the new game**
✅ Create a **demo video script or live stream plan**  
✅ **submit this to IndieGameDev communities**

Just say the word — I'm here to help you ship it!
