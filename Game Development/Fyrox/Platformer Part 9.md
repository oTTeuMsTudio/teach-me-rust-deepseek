We’ve built a solid 2D platformer in **Fyrox** — now it’s time to **polish and publish** it. Let’s walk through the final steps to improve your game and prepare it for release.

---

# 🛠️ Step-by-Step Guide: Improve & Publish Your Fyrox Platformer

---

## 🎯 Goal:
Turn your working prototype into a **shippable game**, with better visuals, performance, and packaging for distribution on platforms like **Windows, macOS, Linux, or even Web (WebAssembly)**.

---

## ✨ Part A: Final Improvements Before Publishing

### 1. **Optimize Performance**

#### ✔️ Reduce Draw Calls
- Use **texture atlases** instead of many small images.
- Combine static tiles into fewer sprites or use `TileMap` if supported.

#### ✔️ Reduce Physics Overhead
- Disable physics for off-screen enemies.
- Use `is_sensor = true` for triggers that don’t need full collision response.

#### ✔️ Optimize Animations
- Don't update animations when not visible.
- Use animation culling or frustum checks.

---

### 2. **Polish Gameplay**

#### ✔️ Add Game Over Screen
Create a new UI canvas named `GameOverScreen`, hidden by default. Show it when health reaches zero.

Add buttons:
- "Retry"
- "Main Menu"

Handle button actions via script messages or direct scene reloading.

#### ✔️ Add Victory Screen
Trigger this when reaching the end of the level.

#### ✔️ Save High Score
Use Rust's `std::fs` or `serde` + JSON to save/load scores:

```rust
use serde::{Serialize, Deserialize};
use std::fs;

#[derive(Serialize, Deserialize)]
struct SaveData {
    high_score: u32,
}

fn save_high_score(score: u32) {
    let data = SaveData { high_score: score };
    fs::write("save.json", serde_json::to_string(&data).unwrap()).expect("Failed to save");
}

fn load_high_score() -> u32 {
    if let Ok(data) = fs::read_to_string("save.json") {
        if let Ok(save) = serde_json::from_str::<SaveData>(&data) {
            return save.high_score;
        }
    }
    0
}
```

Call these from UI or game logic.

---

### 3. **Improve Polish**

#### ✔️ Add Fonts
Use custom fonts for better style:
- `.ttf` or `.otf`
- Load font in script and assign to UI text nodes.

```rust
let font = context.constructor.get_font("data/fonts/PressStart2P.ttf");
if let Some(ui_text) = context.scene.graph.try_get_mut(self.score_text).and_then(|n| n.cast_mut::<Text>()) {
    ui_text.set_font(font);
}
```

#### ✔️ Add Splash Screen
Show a logo screen before gameplay starts.

You can add a `Canvas` node called `SplashScreen`, show it at startup, then hide after a few seconds.

#### ✔️ Add Loading Screen
If you have large scenes, show a loading bar or spinner during transitions.

---

## 📦 Part B: Packaging and Distribution

Now that the game is improved, let’s build and package it.

---

### 1. **Build in Release Mode**

From terminal:
```bash
cargo build --release
```

This builds optimized binaries.

---

### 2. **Run the Game Standalone**

The executable will be in:
```
target/release/platformer
```

Try running it directly:
```bash
./target/release/platformer
```

Make sure all assets are loaded correctly.

---

### 3. **Bundle Assets with Binary**

Ensure all assets (`data/`) are included with the binary.

On Windows/macOS/Linux:
- Copy the entire `data/` folder next to the executable.
- The engine loads assets relative to the executable path.

Structure:
```
platformer.exe
data/
  └── sounds/
  └── textures/
  └── scenes/
```

---

### 4. **Package as ZIP or Installer (Optional)**

#### 🔧 For Windows
Use tools like:
- [NSIS](https://nsis.sourceforge.io/Main_Page) – free installer
- [Inno Setup](https://jrsoftware.org/)
- Or zip everything together

#### 🐧 For Linux
- Package as `.deb`, `.rpm`, or just a `.tar.gz` archive
- Include a `.desktop` launcher file

#### 🍏 For macOS
- Bundle your app using `cargo-bundle`:
```bash
cargo install cargo-bundle
```

Update `Cargo.toml`:
```toml
[package.metadata.bundle]
name = "Platformer"
identifier = "com.yourname.platformer"
icon = ["data/icon.png"]
resources = ["data"]
```

Then run:
```bash
cargo bundle --release --target aarch64-apple-darwin
```

Output: `target/release/bundle/macos/Platformer.app`

---

## 🌐 Part C: Optional - Export to Web (WebAssembly)

Want to share your game online? You can export to **WebAssembly (Wasm)**!

### ✅ Requirements:
- Install `wasm-pack`: https://rustwasm.github.io/wasm-pack/installer/

### ✅ Steps:

1. Set target to wasm32:
```bash
rustup target add wasm32-unknown-unknown
```

2. Build for web:
```bash
cargo build --target wasm32-unknown-unknown --release
```

3. Use `wasm-bindgen` or `fyrox-web` to create an HTML wrapper.

> Note: Fyrox has experimental support for Web. You may need to modify code or wait for official web module support.

Check out:
- [Fyrox Web Example](https://github.com/FyroxEngine/Fyrox/tree/master/examples/web)
- [Fyrox WASM Docs](https://fyrox-book.github.io/fyrox-docs/fyrox_web/index.html)

---

## 🧪 Final Testing Checklist

✅ | Task
---|---
✔️ | Run on all target platforms
✔️ | Test input devices (keyboard, controller if supported)
✔️ | Ensure all audio plays
✔️ | Confirm levels load correctly
✔️ | Check UI scaling on different resolutions
✔️ | Verify save/load works
✔️ | Crash testing – try edge cases

---

## 🚀 Ready to Publish!

Once everything looks good, you’re ready to publish your game!

### Popular Platforms:
| Platform | Link |
|---------|------|
| Itch.io | [itch.io](https://itch.io/) |
| Steam (via Greenlight) | [store.steampowered.com/steamworks/](https://store.steampowered.com/steamworks/) |
| GameJam Sites | [Global Game Jam](https://globalgamejam.org), [Ludum Dare](https://ldjam.com) |
| Personal Website | Host as `.zip` or Web version |

---

## 🧠 Bonus:

- Creating a splash screen
- Adding a main menu
- Supporting controllers/gamepads
- Making a trailer or promotional video
- Setting up social media or itch.io page

Next: **main menu** AND **splash screen** 
