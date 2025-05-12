# Learn Fyrox.rs

---

## 🔍 What is Fyrox?

**Fyrox** (formerly called `rg3d`) is a **3D game engine written in Rust**, designed to be powerful, safe, and fast — all the things Rust does well.

It’s great for making 2D/3D games with modern features like:

- Physics
- Animation
- Audio
- Scripting
- Scene editor (visual UI)
- Cross-platform support (Windows, macOS, Linux)

---

## 🧱 Basic Concepts of Fyrox

Let’s break it down into **simple concepts** before diving into code.

### 1. **Engine**
The core of Fyrox. It handles rendering, physics, audio, etc.

### 2. **Scene**
A scene is like your game world — it contains everything: characters, lights, cameras, terrain, etc.

### 3. **Node**
Everything in a scene is a node. A node can be:
- A camera
- A light
- A mesh (like a cube or character)
- A UI element
- Or even a custom script!

### 4. **Handle<T>**
In Fyrox, you don’t use normal references (`&`), you use `Handle<T>` to refer to nodes and resources. This helps avoid memory issues safely.

### 5. **Resource Manager**
Used to load assets (models, textures, sounds) from disk.

### 6. **Game Loop**
You update your game every frame inside the loop. You can handle input, move objects, check collisions, etc.

---

## 🚀 Let’s Make a Simple App (Hello Cube!)

We'll make a basic app that shows a 3D cube on screen.

### Step 1: Create a New Rust Project

```bash
cargo new hello_fyrox
cd hello_fyrox
```

### Step 2: Add Fyrox to `Cargo.toml`

```toml
[dependencies]
fyrox = "0.37" # Use latest version if needed
```

### Step 3: Write Some Code

Replace contents of `main.rs` with this:

```rust
use fyrox::{
    event_loop::EventLoop,
    engine::{Engine, EngineInitParams},
    scene::{Scene},
    plugin::{Plugin, PluginContext},
    asset::ResourceManager,
    scene::base::BaseBuilder,
    scene::node::Node,
    scene::camera::CameraBuilder,
    scene::mesh::MeshBuilder,
    scene::mesh::shape::CubeBuilder,
    core::pool::Handle,
};

struct Game {
    cube: Handle<Node>,
}

impl Plugin for Game {
    fn init(&mut self, context: PluginContext) {
        // Set up camera
        CameraBuilder::new(BaseBuilder::new()).build(context.scene);

        // Create a red cube
        self.cube = MeshBuilder::new(
            BaseBuilder::new(),
            CubeBuilder::new().build(context.geometry_cache.clone()),
        )
        .with_diffuse_texture(Some("data/textures/red.png".into()))
        .build(context.scene);
    }

    fn update(&mut self, _context: PluginContext) {
        // Optional: Move cube or do logic here
    }
}

fn main() {
    let event_loop = EventLoop::new();
    let mut engine = Engine::new(EngineInitParams::new("Hello Fyrox", &event_loop).unwrap());

    // Load a texture
    let resource_manager: ResourceManager = engine.resource_manager.clone();

    // Add our plugin
    engine.add_plugin(Game { cube: Default::default() });

    // Run the main loop
    event_loop.run(move |event, _, control_flow| {
        engine.update(event, control_flow);
    });
}
```

### Step 4: Add a Texture

Create a folder called `data/textures`, and put a file named `red.png` there. You can use any red image or create one using a paint tool.

---

## 🕹️ Running Your Game

```bash
cargo run
```

If everything works, you’ll see a red cube in a window!

---

## 📚 Learn More (Easy Resources)

- [Fyrox GitHub](https://github.com/FyroxEngine/Fyrox)
- [Fyrox Book (Documentation)](https://docs.fyrox.rs/)
- [YouTube tutorials by the creator](https://www.youtube.com/@MolchDev)
- [Discord Server](https://discord.gg/KkKZYxgNqU) – Ask questions live!

---

## 🧠 Tips to Keep Learning

- Start small: try moving the cube with keyboard input.
- Try loading a `.fbx` or `.glb` model.
- Add lighting and shadows.
- Explore the visual editor once you're comfortable with code.

---

## ✅ Summary

| Concept | Description |
|--------|-------------|
| `Engine` | Main object running everything |
| `Scene` | Container for all game objects |
| `Node` | An object in the scene (cube, camera, etc.) |
| `Handle<Node>` | How you refer to nodes |
| `Plugin` | Where you write your game logic |
| `Resource Manager` | Loads textures/models |
| `Game Loop` | Runs every frame |

---

Next:
- Add player movement
- Load a 3D model
- Add UI
- Use the Fyrox Editor

