Let's continue with **Part 5** of the Fyrox 2D Platformer Tutorial: [Fyrox Book - Platformer Part 5](https://fyrox-book.github.io/tutorials/platformer/part5.html)

---

# 🧭 Fyrox 2D Platformer - Part 5: Level Transitions and Checkpoints

In this part, we’ll:
- Add a **checkpoint system**
- Implement **player respawn**
- Create **level transitions**
- Use **teleporters or level triggers**
- Handle **scene loading/unloading**

Let’s dive in step by step.

---

## 🚪 Step 1: Add a Checkpoint Object

### ✅ Create Checkpoint Node
1. In the editor:
   - Right-click `__ROOT__` → `Add Child → 2D → Rectangle`
   - Set size to `0.5 x 0.5`
   - Set texture to `data/objects/checkpoint.png`
   - Rename it to `Checkpoint`

2. Add a **Trigger Collider**:
   - Right-click the checkpoint node → `Add Child → Physics2D → Collider`
   - Shape: `Cuboid`, Size: `0.5 x 0.5`
   - Enable `Is Trigger` checkbox

---

## 🧠 Step 2: Create Checkpoint Script

### ✅ Define `Checkpoint` Script Struct

```rust
#[derive(Visit, Reflect, Debug, Clone, Default, TypeUuidProvider)]
#[type_uuid(id = "b1c2d3e4-f5a6-7890-g1h2-i3j4k5l6m7n8")]
#[visit(optional)]
struct Checkpoint {
    is_activated: bool,
}
```

### 🎮 Implement `ScriptTrait` for Checkpoint

```rust
impl ScriptTrait for Checkpoint {
    fn on_init(&mut self, context: &mut ScriptContext) {}

    fn on_start(&mut self, _context: &mut ScriptContext) {}

    fn on_update(&mut self, _context: &mut ScriptContext) {}

    fn on_contact(
        &mut self,
        context: &mut ScriptContext,
        this: Handle<Node>,
        contact_data: &ContactData,
    ) {
        if contact_data.active_contacts > 0 && !self.is_activated {
            // Broadcast message that player activated a checkpoint
            context.message_sender.send(ScriptMessage {
                receiver: context.handle,
                message: CheckpointMessage::Activated(this),
            });

            self.is_activated = true;
        }
    }
}
```

### ⚙️ Register the Script
Inside your plugin:

```rust
script_constructors.add::<Checkpoint>("Checkpoint");
```

---

## 📦 Step 3: Store Checkpoint Position in Player

### ✅ Add Fields to `Player` Struct

```rust
#[derive(Debug, Clone, Copy)]
struct CheckpointData {
    position: Vector2<f32>,
}

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
}
```

### 🔄 Update `on_message()` in Player Script

Add this method to handle activation:

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
}
```

Define the message enum:

```rust
#[derive(Serialize, Deserialize, Debug)]
enum CheckpointMessage {
    Activated(Handle<Node>),
}
```

---

## 🛏️ Step 4: Implement Respawn Logic

### ✅ Add Method to `Player`

```rust
fn die(&mut self, context: &mut ScriptContext) {
    if let Some(ref checkpoint) = self.checkpoint {
        if let Some(rigid_body) = context.scene.graph[context.handle].cast_mut::<RigidBody>() {
            rigid_body.set_pos(Vector3::new(checkpoint.position.x, checkpoint.position.y, 0.0));
            rigid_body.set_lin_vel(Vector2::new(0.0, 0.0));
        }
    } else {
        // No checkpoint — reset to start
        if let Some(rigid_body) = context.scene.graph[context.handle].cast_mut::<RigidBody>() {
            rigid_body.set_pos(Vector3::new(0.0, 0.0, 0.0));
            rigid_body.set_lin_vel(Vector2::new(0.0, 0.0));
        }
    }

    println!("Player respawned!");
}
```

This will teleport the player back to the last checkpoint (or start).

---

## 🚪 Step 5: Add Level Transition Trigger

### ✅ Create Level Trigger Node
1. Add a new `Rectangle` node.
2. Add a **Trigger Collider**.
3. Attach a script to detect when the player enters.

### 🎮 Define LevelTrigger Script

```rust
#[derive(Visit, Reflect, Debug, Clone, Default, TypeUuidProvider)]
#[type_uuid(id = "d2e3f4a5-b6c7-8901-d2e3-f4a5b6c78901")]
#[visit(optional)]
struct LevelTrigger {
    target_scene: String,
}

impl ScriptTrait for LevelTrigger {
    fn on_init(&mut self, context: &mut ScriptContext) {}

    fn on_start(&mut self, _context: &mut ScriptContext) {}

    fn on_update(&mut self, _context: &mut ScriptContext) {}

    fn on_contact(
        &mut self,
        context: &mut ScriptContext,
        _this: Handle<Node>,
        contact_data: &ContactData,
    ) {
        if contact_data.active_contacts > 0 {
            context.plugin_controlled_scene = Some(self.target_scene.clone());
            context.scene_loader.request_new_scene(self.target_scene.clone(), true);
        }
    }
}
```

Register the script:

```rust
script_constructors.add::<LevelTrigger>("LevelTrigger");
```

Set `target_scene` in the inspector to something like `"levels/level2.scene"`.

---

## 🧪 Step 6: Test It!

Hit **Play** and test:
- Walk into checkpoint → should activate
- Kill player → should respawn at checkpoint
- Walk into trigger → should load next scene

---

## 📝 Summary of What We Did in Part 5

| Feature | Description |
|--------|-------------|
| Checkpoints | Added visual checkpoint object and logic |
| Respawning | Teleports player back after death |
| Message System | Used `on_message` to communicate between scripts |
| Level Triggers | Implemented basic level transition system |
| Scene Loading | Switch scenes using `SceneLoader` |

---

Continue with **Part 6**, where we add:
- **UI (health bar, score display)**
- **HUD elements**
- **Text rendering**
- **Pause menu**
