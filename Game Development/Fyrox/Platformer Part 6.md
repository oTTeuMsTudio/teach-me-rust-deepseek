Let's continue with **Part 6** of the Fyrox 2D Platformer Tutorial: [Fyrox Book - Platformer Part 6](https://fyrox-book.github.io/tutorials/platformer/part6.html)

---

# 🧾 Fyrox 2D Platformer - Part 6: UI and HUD

In this part, we’ll:
- Add a **UI overlay**
- Display **player health**
- Show **score/coins collected**
- Create a **pause menu**
- Use **text rendering in 2D**

Let’s go step by step.

---

## 🖼️ Step 1: Create UI Root Node

### ✅ Add UI Camera
1. In the editor:
   - Right-click `__ROOT__` → `Add Child → Camera`
   - Set `Clear Color` to `None`
   - Set `Projection` to `Orthographic`
   - Set `Size` to `10.0`
   - Rename it to `UICamera`

This camera will render only UI elements.

---

## 💬 Step 2: Add Text for Health and Score

### ✅ Add UI Root Node
Right-click `__ROOT__` → `Add Child → UI → Canvas`

Name it `HUD`.

Inside `HUD`, add two `Text` nodes:
- `HealthText`
- `ScoreText`

### ⚙️ Configure Text Nodes
Select each text node and set:
- Content: e.g., `"Health: 3"`
- Font Size: `24`
- Horizontal/Vertical Alignment: `Left` / `Top`
- Margin: `(10, 10, 0, 0)` for `HealthText`, `(10, 50, 0, 0)` for `ScoreText`

You can also change color and font if needed.

---

## 🧠 Step 3: Update Player Script to Track Health and Score

### ✅ Add Fields to `Player` Struct

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
    current_state: PlayerAnimationState,
    is_grounded: bool,
    can_jump: bool,
    checkpoint: Option<CheckpointData>,
    health: u32,
    score: u32,
    health_text: Handle<Node>,
    score_text: Handle<Node>,
}
```

### 🔄 Load Text Handles in `on_start()`

```rust
fn on_start(&mut self, context: &mut ScriptContext) {
    // Existing code...

    self.health = 3;
    self.score = 0;

    // Find UI text nodes
    self.health_text = find_ui_node(context, "HealthText");
    self.score_text = find_ui_node(context, "ScoreText");

    update_ui(context, self.health_text, format!("Health: {}", self.health));
    update_ui(context, self.score_text, format!("Score: {}", self.score));
}

// Helper functions
fn find_ui_node(context: &mut ScriptContext, name: &str) -> Handle<Node> {
    context.scene.graph.find_by_name_from_root(name).map(|n| n.1).unwrap_or_default()
}

fn update_ui(context: &mut ScriptContext, handle: Handle<Node>, text: String) {
    if let Some(ui_text) = context.scene.graph.try_get_mut(handle) {
        if let Some(text) = ui_text.cast_mut::<fyrox::scene::ui::Text>() {
            text.set_content(text);
        }
    }
}
```

---

## 💔 Step 4: Update Health When Taking Damage

### ✅ Modify `die()` Method to Decrease Health

```rust
fn die(&mut self, context: &mut ScriptContext) {
    self.health -= 1;

    if self.health > 0 {
        // Respawn
        if let Some(ref checkpoint) = self.checkpoint {
            if let Some(rigid_body) = context.scene.graph[context.handle].cast_mut::<RigidBody>() {
                rigid_body.set_pos(Vector3::new(checkpoint.position.x, checkpoint.position.y, 0.0));
                rigid_body.set_lin_vel(Vector2::new(0.0, 0.0));
            }
        } else {
            if let Some(rigid_body) = context.scene.graph[context.handle].cast_mut::<RigidBody>() {
                rigid_body.set_pos(Vector3::new(0.0, 0.0, 0.0));
                rigid_body.set_lin_vel(Vector2::new(0.0, 0.0));
            }
        }

        update_ui(context, self.health_text, format!("Health: {}", self.health));
    } else {
        // Game over
        println!("Game Over!");
    }
}
```

---

## 🪙 Step 5: Add Coin Object and Collect Logic

### ✅ Create Coin Node
1. Add a new `Rectangle` node.
2. Assign texture: `data/objects/Coin.png`.
3. Add a **Trigger Collider**.

### 🎮 Create Coin Script

```rust
#[derive(Visit, Reflect, Debug, Clone, Default, TypeUuidProvider)]
#[type_uuid(id = "e5f6a7b8-c9d0-1234-e5f6-a7b8c9d01234")]
#[visit(optional)]
struct Coin {}

impl ScriptTrait for Coin {
    fn on_init(&mut self, _context: &mut ScriptContext) {}

    fn on_start(&mut self, _context: &mut ScriptContext) {}

    fn on_update(&mut self, _context: &mut ScriptContext) {}

    fn on_contact(
        &mut self,
        context: &mut ScriptContext,
        this: Handle<Node>,
        contact_data: &ContactData,
    ) {
        if contact_data.active_contacts > 0 {
            context.message_sender.send(ScriptMessage {
                receiver: context.handle,
                message: CoinMessage::Collected(this),
            });

            context.scene.graph.remove_node(this);
        }
    }
}
```

Define message:

```rust
#[derive(Serialize, Deserialize, Debug)]
enum CoinMessage {
    Collected(Handle<Node>),
}
```

Register script:

```rust
script_constructors.add::<Coin>("Coin");
```

### 🔄 Handle Coin Collection in Player Script

Update `on_message()`:

```rust
fn on_message(&mut self, context: &mut ScriptContext, message: &[u8]) {
    if let Ok(msg) = bincode::deserialize::<CheckpointMessage>(message) {
        match msg {
            CheckpointMessage::Activated(checkpoint_node) => {
                if let Some(node) = context.scene.graph.try_get(checkpoint_node) {
                    let pos = node.global_position();
                    self.checkpoint = Some(CheckpointData {
                        position: Vector2::new(pos.x, pos.y),
                    });
                    println!("Checkpoint activated at {:?}", pos);
                }
            }
        }
    }

    if let Ok(msg) = bincode::deserialize::<CoinMessage>(message) {
        match msg {
            CoinMessage::Collected(_) => {
                self.score += 1;
                update_ui(context, self.score_text, format!("Score: {}", self.score));
            }
        }
    }
}
```

---

## 🛑 Step 6: Add Pause Menu (Optional)

### ✅ Create Pause Menu UI

In the editor:
- Add a new `Canvas` node named `PauseMenu`
- Inside it, add a `Text` node saying `"Paused"` and a `Button` node labeled `"Resume"`

Initially, hide the pause menu (`Visible = false`)

### 🎮 Add Input Handling for Pause

Update `on_os_event()` in `Player`:

```rust
fn on_os_event(&mut self, event: &Event<()>, context: &mut ScriptContext) {
    if let Event::WindowEvent { event, .. } = event {
        if let WindowEvent::KeyboardInput { event, .. } = event {
            if let PhysicalKey::Code(keycode) = event.physical_key {
                let is_pressed = event.state == ElementState::Pressed;
                match keycode {
                    KeyCode::KeyA => self.move_left = is_pressed,
                    KeyCode::KeyD => self.move_right = is_pressed,
                    KeyCode::Space => self.jump = is_pressed,
                    KeyCode::Escape => {
                        if is_pressed {
                            toggle_pause_menu(context, true);
                        }
                    }
                    _ => {}
                }
            }
        }
    }
}
```

Helper function:

```rust
fn toggle_pause_menu(context: &mut ScriptContext, visible: bool) {
    if let Some(pause_menu) = context.scene.graph.find_by_name_from_root("PauseMenu").map(|(_, h)| h) {
        context.scene.graph[pause_menu].set_visibility(visible);
    }
}
```

---

## 🧪 Step 7: Test It!

Hit **Play** and test:
- Collect coins → score updates
- Die → health decreases and respawns
- Activate checkpoints → respawn point updated
- Press `Esc` → pause menu appears

---

## 📝 Summary of What We Did in Part 6

| Feature | Description |
|--------|-------------|
| UI System | Created HUD using `Canvas` and `Text` |
| Health Display | Shows remaining player lives |
| Score Display | Tracks coins collected |
| Coin Pickup | Added coin object and collection logic |
| Pause Menu | Implemented basic pause screen |

---

Continue with **Part 7**, where we add:
- **Sound effects**
- **Music**
- **Audio triggers (jump, death, collect)**
