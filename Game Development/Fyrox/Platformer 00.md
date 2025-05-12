# Let's walk through creating a 2D platformer game using the **Fyrox engine** step by step, 

following the [Fyrox Book - Platformer Part 1](https://fyrox-book.github.io/tutorials/platformer/part1.html) tutorial. This will be very detailed and precise.

---

## 🧱 Step 1: Set Up Your Project

### ✅ Install `fyrox-template`
This tool generates Fyrox projects with all boilerplate code:
```bash
cargo install fyrox-template
```

### ✅ Create the Project
Navigate to your desired folder and run:
```bash
fyrox-template init --name platformer --style 2d
```

### ✅ Navigate into the project
```bash
cd platformer
```

### ✅ Run the Editor (Development Mode)
```bash
cargo run --package editor --release
```
You'll see the Fyrox editor GUI open.

---

## 🎨 Step 2: Prepare Assets and Scene

### 💾 Download Tutorial Assets
From the tutorial page, download the asset pack and extract it into the `data` folder of your project root.

### 🚪 Open the Editor and Clean the Scene
- Click on `File > New Scene` to create a new scene.
- Remove any default content from the scene (`World Viewer` panel).

### 📐 Switch Camera to 2D Mode
In the top toolbar in the scene preview window, click the camera mode and select `2D`.

---

## 🌄 Step 3: Create Ground Platforms

### ⬛ Add Static Rigid Body for Ground
1. Right-click `__ROOT__` in `World Viewer`.
2. Select `Add Child → Physics2D → Rigid Body`.
3. Set `Body Type` to **Static** in the `Inspector`.

### 🔳 Add Collider to the Rigid Body
1. Right-click the rigid body in the `World Viewer`.
2. Select `Add Child → Physics2D → Collider`.
3. In Inspector:
   - Shape → `Cuboid`
   - Size → `1.0 x 1.0`

### 🖼️ Add Sprite to the Rigid Body
1. Right-click the rigid body again.
2. Select `Add Child → 2D → Rectangle`.
3. In Inspector:
   - Open `Material`
   - Drag and drop texture from `Asset Browser`:
     - For example, use `data/tiles/14.png` as center ground block.
   - Adjust UV Rect if needed.

### ➕ Clone and Arrange Ground Blocks
- Press `Ctrl+C` and `Ctrl+V` to clone the entire block.
- Change textures to `13.png` and `15.png` for left/right ends.
- Use `Move Tool` (shortcut `W`) to position them side-by-side.
- Optionally enable grid snapping via `File > Settings > Interaction > Snap To Grid`.

Repeat this process to build more platforms or obstacles.

---

## 🏙️ Step 4: Add Background

### 📷 Add a Background Image
1. Right-click `__ROOT__` in `World Viewer`.
2. Select `Add Child → 2D → Rectangle`.
3. Assign `data/background/BG.png` to its material.
4. Scale and position it behind everything else.
5. You can adjust Z-order by reordering nodes under `__ROOT__`.

---

## 📦 Step 5: Add Dynamic Objects (e.g., Boxes)

### 📦 Create Dynamic Box
1. Duplicate one of the static ground blocks.
2. Change the `Body Type` of the rigid body to **Dynamic**.
3. Replace sprite texture with `data/objects/Crate.png`.
4. Clone this box multiple times and place them on platforms.

---

## 👤 Step 6: Create Player Character

### 🧠 Add Rigid Body for Player
1. Right-click `__ROOT__` → `Add Child → Physics2D → Rigid Body`.
2. Set `Body Type` to **Dynamic**.

### 🥛 Add Collider (Capsule Shape)
1. Add a `Collider` child node.
2. Set shape to **Capsule**.
3. Parameters:
   - Begin = `0.0, 0.0`
   - End = `0.0, 0.3`

### 🎭 Add Player Sprite
1. Add a `Rectangle` node as child of rigid body.
2. Set texture to `data/characters/adventurer/adventurer-Sheet.png`.
3. Set UV Rect to `(0.0, 0.0, 0.143, 0.091)` to display first frame.

### 📸 Add Camera
1. Add `Camera` node as child of player rigid body.
2. In Inspector:
   - Skybox → `Some`
   - Front → Drag `data/background/BG.png`
   - Far Plane → Set to `20.0`

---

## 💾 Save the Scene
Click `File → Save Scene` and save it as something like `main.scene`.

---

## 🧠 Step 7: Write Player Script Logic

### 🧩 Import Required Modules
At the top of `game/src/lib.rs`, add:
```rust
use fyrox::{
    core::algebra::{Vector2, Vector3},
    event::{ElementState, Event, WindowEvent},
    keyboard::KeyCode,
    scene::dim2::rigidbody::RigidBody,
    script::{ScriptContext, ScriptTrait},
    scene::node::Node,
    scene::animation::spritesheet::SpriteSheetAnimation,
    scene::dim2::rectangle::Rectangle,
};
```

### 🎮 Define Player Script Struct
Replace or append at bottom of file:
```rust
#[derive(Visit, Reflect, Debug, Clone, Default, TypeUuidProvider)]
#[type_uuid(id = "c5671d19-9f1a-4286-8486-add4ebaadaec")]
#[visit(optional)]
struct Player {
    move_left: bool,
    move_right: bool,
    jump: bool,
    sprite: Handle<Node>,
    animations: Vec<SpriteSheetAnimation>,
    current_animation: u32,
}
```

### 🔄 Implement `ScriptTrait`
```rust
impl ScriptTrait for Player {
    fn on_init(&mut self, context: &mut ScriptContext) {}

    fn on_start(&mut self, context: &mut ScriptContext) {}

    fn on_os_event(&mut self, event: &Event<()>, context: &mut ScriptContext) {
        if let Event::WindowEvent { event, .. } = event {
            if let WindowEvent::KeyboardInput { event, .. } = event {
                if let PhysicalKey::Code(keycode) = event.physical_key {
                    let is_pressed = event.state == ElementState::Pressed;
                    match keycode {
                        KeyCode::KeyA => self.move_left = is_pressed,
                        KeyCode::KeyD => self.move_right = is_pressed,
                        KeyCode::Space => self.jump = is_pressed,
                        _ => {}
                    }
                }
            }
        }
    }

    fn on_update(&mut self, context: &mut ScriptContext) {
        if let Some(rigid_body) = context.scene.graph[context.handle].cast_mut::<RigidBody>() {
            let x_speed = if self.move_left {
                -3.0
            } else if self.move_right {
                3.0
            } else {
                0.0
            };

            if self.jump {
                rigid_body.set_lin_vel(Vector2::new(x_speed, 4.0));
            } else {
                rigid_body.set_lin_vel(Vector2::new(x_speed, rigid_body.lin_vel().y));
            }

            // Flip sprite based on direction
            if let Some(sprite_node) = context.scene.graph.try_get_mut(self.sprite) {
                if x_speed != 0.0 {
                    let transform = sprite_node.local_transform_mut();
                    let scale = **transform.scale();
                    transform.set_scale(Vector3::new(
                        current_scale.x.copysign(-x_speed),
                        scale.y,
                        scale.z,
                    ));
                }
            }

            // Animation logic goes here (see next step)
        }
    }
}
```

### 🧪 Register the Script
In the `Plugin` implementation, inside `register()`:
```rust
script_constructors.add::<Player>("Player");
```

---

## 🎞️ Step 8: Add Animations

### 🧠 Update Player Fields
Make sure you have:
```rust
animations: Vec<SpriteSheetAnimation>,
current_animation: u32,
```

### 🧩 Initialize Animations in `on_start`
Inside `on_start()`:
```rust
fn on_start(&mut self, context: &mut ScriptContext) {
    self.animations = vec![
        SpriteSheetAnimation::from_definition(
            context.scene.graph[self.sprite]
                .query_component_mut::<Rectangle>()
                .unwrap()
                .material(),
            "run",
            0.1,
            true,
        ),
        SpriteSheetAnimation::from_definition(
            context.scene.graph[self.sprite]
                .query_component_mut::<Rectangle>()
                .unwrap()
                .material(),
            "idle",
            0.5,
            true,
        ),
    ];
}
```

### 🔄 Update Animation in `on_update`
Inside `on_update()` after movement:
```rust
if x_speed != 0.0 {
    self.current_animation = 1; // Run
} else {
    self.current_animation = 0; // Idle
}

if let Some(current_animation) = self.animations.get_mut(self.current_animation as usize) {
    current_animation.update(context.dt);

    if let Some(sprite) = context
        .scene
        .graph
        .try_get_mut(self.sprite)
        .and_then(|n| n.cast_mut::<Rectangle>()) {
        sprite
            .material()
            .data_ref()
            .bind("diffuseTexture", current_animation.texture());
        sprite.set_uv_rect(current_animation.current_frame_uv_rect().unwrap_or_default());
    }
}
```

---

## 🧪 Step 9: Assign Script in Editor

### 🧑‍💻 Back in the Editor:
1. Select the **player's rigid body** in the `World Viewer`.
2. Go to `Inspector > Script > +` and choose `Player`.
3. Drag the **Rectangle (sprite)** node into the `sprite` field.

### 🎞️ Add Animations in Inspector
1. Expand the `Animations` array.
2. Add two entries:
   - First for `run` animation: set frames and time per frame.
   - Second for `idle`.

---

## ▶️ Step 10: Test Your Game

- Hit the **Play** button in the editor.
- The player should:
  - Move left/right with `A/D`
  - Jump with `Space`
  - Animate idle/run
  - Face correct direction

---

## 🧹 Final Notes

- You now have a working 2D platformer character.
- You can expand this by adding enemies, collectibles, levels, UI, etc.
- Future parts of the tutorial will cover advanced topics like AI, level loading, and polish.

Continuing to part 2 or solving issues during setup!
