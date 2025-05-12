# We're ready to go deeper. Let's pick one of the next steps and learn it **step-by-step** in a simple way.

---

## 🎯 What would you like to learn now?

Pick one (or I can guide you through all over time):

1. **[Player Movement](#player-movement)** – Move the cube with keyboard
2. **[Load 3D Model](#load-3d-model)** – Load `.glb` or `.fbx` models
3. **[Add UI](#add-ui)** – Add text, buttons, health bars
4. **[Use Fyrox Editor](#use-fyrox-editor)** – Visual scene editor

---

## 1. 🕹️ Player Movement

Let’s make that cube move around using WASD keys!

### 🔧 Update Your `Game` Plugin

We’ll modify the `update()` method to check for input and move the cube.

```rust
use fyrox::{
    event_loop::EventLoop,
    engine::{Engine, EngineInitParams},
    plugin::{Plugin, PluginContext},
    scene::{Scene, node::Node, camera::CameraBuilder, transform::TransformBuilder},
    scene::mesh::{MeshBuilder, shape::CubeBuilder},
    core::{vector::Vector3, pool::Handle},
    event::{DeviceEvent, ElementState, VirtualKeyCode},
};

struct Game {
    cube: Handle<Node>,
}

impl Plugin for Game {
    fn init(&mut self, context: PluginContext) {
        // Camera
        CameraBuilder::new(BaseBuilder::new()).build(context.scene);

        // Cube
        self.cube = MeshBuilder::new(
            BaseBuilder::new(),
            CubeBuilder::new().build(context.geometry_cache.clone()),
        )
        .with_local_transform(
            TransformBuilder::new()
                .with_local_position(Vector3::new(0.0, 0.0, 5.0)) // place cube in front of camera
                .build()
        )
        .build(context.scene);
    }

    fn update(&mut self, context: PluginContext) {
        let speed = 0.1;

        if let Some(input) = context.input {
            let mut transform = context.scene.graph[self.cube].local_transform_mut();

            if input.key_is_down(VirtualKeyCode::W) {
                transform.set_position(transform.position() + Vector3::new(0.0, 0.0, -speed));
            }
            if input.key_is_down(VirtualKeyCode::S) {
                transform.set_position(transform.position() + Vector3::new(0.0, 0.0, speed));
            }
            if input.key_is_down(VirtualKeyCode::A) {
                transform.set_position(transform.position() + Vector3::new(-speed, 0.0, 0.0));
            }
            if input.key_is_down(VirtualKeyCode::D) {
                transform.set_position(transform.position() + Vector3::new(speed, 0.0, 0.0));
            }
        }
    }
}
```

### ✅ What This Does:

- Checks for key presses (`WASD`)
- Moves the cube forward/back/left/right
- Uses `local_transform_mut()` to change position

---

## 2. 📦 Load 3D Model (.glb or .fbx)

Fyrox supports loading models from `.glb`, `.gltf`, `.fbx`, `.obj`, etc.

### Step 1: Put a Model in Your Project

Place a model file like `data/models/robot.glb`.

You can download free GLB models from [https://sketchfab.com](https://sketchfab.com) or use [Google Poly](https://poly.google.com).

### Step 2: Load It in Code

```rust
let resource_manager = context.resource_manager.clone();
let model = resource_manager.request_model("data/models/robot.glb").unwrap();

// When model is loaded, spawn it into the scene
model.into_future().then(|result| {
    let model = result.unwrap();
    let scene = &mut context.scene;
    scene.instantiate_model(&model);
});
```

> ⚠️ Loading assets is asynchronous, so we use `.into_future()` to wait until it's done.

---

## 3. 🖼️ Add UI (Text, Buttons, Health Bars)

Fyrox has a powerful UI system built on `fyrox-ui`.

### Step 1: Add UI Dependency

In `Cargo.toml`:

```toml
[dependencies]
fyrox = "0.37"
fyrox-ui = "0.10" # or latest version
```

### Step 2: Add Text UI

```rust
use fyrox_ui::{
    UiRoot, TextBuilder, HorizontalAlignment, VerticalAlignment, UserInterface,
};
use fyrox_ui::text::TextMessage;

impl Plugin for Game {
    fn init(&mut self, context: PluginContext) {
        // Create UI Root
        let ui_root = UiRoot::new();

        // Add some text
        TextBuilder::new(fyrox_ui::widget::WidgetBuilder::new())
            .with_text("Hello, Fyrox UI!")
            .build(&mut ui_root.ui.build_ctx());

        // Set UI as root
        context.engine.set_ui(ui_root);
    }
}
```

You can later add:
- Buttons
- Health bars (progress bars)
- Inventory panels

---

## 4. 🧰 Use Fyrox Editor

The Fyrox Editor lets you visually build your game scenes — no code needed for placing objects, lights, or cameras.

### Step 1: Install the Editor

```bash
cargo install fyrox-tools
```

### Step 2: Run the Editor

```bash
fyrox-ed
```

### Step 3: Create a Scene

- Add a camera
- Add a light
- Add a cube or model
- Save the scene as `scene.rgs`

### Step 4: Load Scene in Code

```rust
let scene_path = "data/scenes/scene.rgs";
let resource_manager = context.resource_manager.clone();
let future = resource_manager.request_model(scene_path);

future.then(|result| {
    let model = result.unwrap();
    context.scene.instantiate_model(&model);
});
```

Now your visual scene appears in-game!

---

## 🔄 Next Steps Roadmap

Here’s how you can progress:

1. ✅ Learn movement
2. ✅ Load 3D models
3. ✅ Build UI
4. ✅ Use editor to design levels
5. ➕ Add physics (jumping, collisions)
6. ➕ Script enemies and AI
7. 🚀 Make a full game!

---

## Next:
- How to add jumping
- How to shoot bullets
- How to play sound effects
- How to do multiplayer

Next: **movement**, **loading a model**
