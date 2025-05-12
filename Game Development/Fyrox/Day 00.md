# Start game development in Rust using the `fyrox` engine

---

## 🧱 What is Fyrox?

Fyrox (formerly called `rg3d`) is a powerful, open-source 3D game engine written in Rust. It features:

- Scene graph with physics integration
- PBR rendering pipeline
- Animation system
- UI framework (also written in Rust)
- Asset loading (glTF, FBX, textures, audio, etc.)
- Cross-platform support (Windows, macOS, Linux)

It's designed for flexibility and performance, making it a great choice for indie developers working in Rust.

---

## 🚀 Prerequisites

Before we begin, ensure you have the following installed:

1. **Rust**: Install from [https://rustup.rs](https://rustup.rs)
2. **A code editor**: VS Code or JetBrains RustRover are good choices.
3. **Git**: Required for some asset tools (optional but recommended).

---

## 📦 Step 1: Create a New Rust Project

Open your terminal and run:

```bash
cargo new my_fyrox_game
cd my_fyrox_game
```

Then add `fyrox` to your `Cargo.toml`:

```toml
[dependencies]
fyrox = "0.36"  # Or check latest version on crates.io
```

> ⚠️ Make sure to use the correct version. You can find the latest at [crates.io/crates/fyrox](https://crates.io/crates/fyrox)

---

## 🧪 Step 2: Minimal Working Example

Replace the contents of `src/main.rs` with this minimal example that creates a window and adds a 3D cube:

```rust
use fyrox::{
    core::{color::Color, pool::Handle},
    engine::{resource_manager::ResourceManager, Engine, SerializationContext},
    event_loop::EventLoop,
    scene::{
        base::BaseBuilder,
        camera::CameraBuilder,
        mesh::MeshBuilder,
        node::Node,
        transform::TransformBuilder,
        Scene,
    },
    plugin::{Plugin, PluginContext, PluginRegistrationContext},
};

struct Game {
    scene: Handle<Scene>,
}

impl Plugin for Game {
    fn on_register(&mut self, _context: PluginRegistrationContext) {
        // Called once during registration
    }

    fn init(&mut self, context: PluginContext) {
        // Setup your game here
        let mut scene = Scene::new();

        // Add a camera
        CameraBuilder::new(
            BaseBuilder::new().with_local_transform(
                TransformBuilder::new()
                    .with_local_position([0.0, 5.0, -10.0].into())
                    .build(),
            ),
        )
        .build(&mut scene.graph);

        // Add a cube
        MeshBuilder::new(BaseBuilder::new())
            .with_cast_shadows(true)
            .build(&mut scene.graph);

        self.scene = context.scenes.add(scene);
    }

    fn update(&mut self, _context: PluginContext) {
        // Game logic goes here
    }
}

fn main() {
    let event_loop = EventLoop::new();
    let graphics_context = fyrox::platform::GraphicsContext::new(event_loop.create_window("My Fyrox Game", Default::default()), None).unwrap();
    
    let serialization_context = Arc::new(SerializationContext::new());
    let mut engine = Engine::new(graphics_context, serialization_context).unwrap();

    let game = Game {
        scene: Default::default(),
    };

    engine.run(game);
}
```

---

## ▶️ Step 3: Run Your Game

In your terminal:

```bash
cargo run
```

You should see a window titled “My Fyrox Game” with a cube rendered in 3D space!

---

## 🎮 Controls & Interaction

By default, Fyrox provides basic controls via the keyboard/mouse:

- **WASD + Mouse Look**: Move and look around (camera controller not included by default yet)
- **Mouse Wheel**: Zoom in/out

To get full navigation, you can use the built-in `orbit camera controller`, which we'll cover next.

---

## 🕹️ Step 4: Add Orbit Camera Controller

Add this inside your `init()` method after creating the camera:

```rust
use fyrox::scene::camera::Projection;
use fyrox::gui::UserInterface;

// Replace the camera creation with:
let camera_handle = CameraBuilder::new(
    BaseBuilder::new()
        .with_local_transform(
            TransformBuilder::new()
                .with_local_position([0.0, 5.0, -10.0].into())
                .build(),
        )
).with_projection(Projection::perspective_fov(std::f32::consts::FRAC_PI_3, 1.333, 0.1, 1000.0))
.build(&mut scene.graph);

// Enable orbit controller
engine.user_interfaces.first_mut().send_ui_message(
    fyrox::gui::message::UiMessage::camera(
        camera_handle,
        fyrox::gui::camera::CameraMessage::orbit_mode(true),
    )
);
```

Now you can move the camera with mouse drag + scroll wheel!

---

## 🖼️ Step 5: Load a Model (e.g., glTF)

### Add a model file

Place a `.gltf` or `.fbx` model in your project folder under `data/models/`.

Example: `data/models/sphere.gltf`

### Load the model in your scene

Inside `init()`:

```rust
let resource_manager = context.serialization_context.resource_manager.clone();

resource_manager.request_model("data/models/sphere.gltf").await.unwrap();
let model = resource_manager.model("data/models/sphere.gltf").unwrap();

model.instantiate(&mut scene);
```

You’ll need to `await` the load — so wrap your `init()` logic in an async block if needed.

---

## 📁 Asset Pipeline Tips

Fyrox supports many assets:

| Type       | Supported Formats         |
|------------|---------------------------|
| Models     | glTF 2.0, FBX             |
| Textures   | PNG, JPEG, DDS            |
| Audio      | WAV, MP3                  |
| Animations | Embedded in glTF/FBX      |

Use `fyrox::plugin::PluginContext::serialization_context.resource_manager` to load them.

---

## 🧰 Tools

Fyrox has its own editor: [Fyroxed](https://github.com/FyroxEngine/Fyroxed)

- Drag-and-drop scene editing
- Visual scripting (coming soon)
- Material editor
- Animation timeline

You can build scenes visually and load them in your game like this:

```rust
let scene = context.serialization_context.resource_manager.request_scene("data/scenes/level.rgs");
```

---

## 🧩 Plugins System

Fyrox uses a **plugin architecture**, allowing you to modularize your game logic.

Each plugin can manage:
- Scene updates
- Input handling
- Resource loading
- GUI elements

This helps scale large projects cleanly.

---

## 🧑‍💻 Community & Resources

- **GitHub**: https://github.com/FyroxEngine/Fyrox
- **Discord**: https://discord.gg/ZWksm2Kymw
- **Examples**: https://github.com/FyroxEngine/Fyrox/tree/master/examples
- **Documentation**: https://docs.fyrox.rs/

---

## ✅ Summary Checklist

| Task                        | Status |
|----------------------------|--------|
| Set up Rust environment    | ✅     |
| Created Cargo project      | ✅     |
| Added `fyrox` as dependency| ✅     |
| Built a simple 3D scene    | ✅     |
| Added orbit camera         | ✅     |
| Loaded a model             | ✅     |
| Explored plugins           | ✅     |

---

## 🧠 Next Steps

Once comfortable with basics, try:

- Adding physics with `nphysics` (integrated into Fyrox)
- Creating a GUI HUD (using fyrox’s built-in UI)
- Implementing player movement/controller
- Loading levels from editor exports
- Multiplayer networking with `leafwing` or `bevy-netcode`

---

Would you like me to create a **template project** structure for a Fyrox game? Or help you make a **simple FPS controller**, **UI menu**, or **enemy AI**?
