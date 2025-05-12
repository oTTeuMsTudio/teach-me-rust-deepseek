Moving fast! Let's continue with **Part 4** of the Fyrox 2D Platformer Tutorial: [Fyrox Book - Platformer Part 4](https://fyrox-book.github.io/tutorials/platformer/part4.html)

---

# 👾 Fyrox 2D Platformer - Part 4: Enemies and Collision Handling

In this part, we will:
- Add an **enemy character**
- Implement **collision detection**
- Handle **player death**
- Use **script messages** to communicate between scripts
- Trigger **enemy AI behavior**

Let’s go step by step.

---

## 🧱 Step 1: Prepare Enemy Assets in Scene

### ✅ Add Enemy Rigid Body
1. In Fyrox Editor:
   - Right-click `__ROOT__` → `Add Child → Physics2D → Rigid Body`
   - Set `Body Type` to **Static** (for now)
2. Add a **Collider** child node:
   - Shape: `Cuboid`, Size: `0.5 x 0.5`
3. Add a **Sprite** child node:
   - Texture: `data/characters/slime/slime.png`
   - UV Rect: `(0.0, 0.0, 1.0, 1.0)`

### 🔄 Later: Switch to Dynamic for Movement
We'll make it dynamic later when adding movement logic.

---

## 🧠 Step 2: Create Enemy Script

### ✅ Define Enemy Script Struct

Add this near the top of `game/src/lib.rs`:

```rust
use fyrox::scene::animation::spritesheet::SpriteSheetAnimation;
```

Then define the enemy script struct:

```rust
#[derive(Visit, Reflect, Debug, Clone, Default, TypeUuidProvider)]
#[type_uuid(id = "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8")]
#[visit(optional)]
struct Enemy {
    sprite: Handle<Node>,
    animation: SpriteSheetAnimation,
}
```

### 🎮 Implement `ScriptTrait` for Enemy

```rust
impl ScriptTrait for Enemy {
    fn on_init(&mut self, context: &mut ScriptContext) {}

    fn on_start(&mut self, context: &mut ScriptContext) {
        if let Some(sprite_node) = context.scene.graph.try_get_mut(self.sprite) {
            if let Some(sprite_rect) = sprite_node.cast_mut::<Rectangle>() {
                let material = sprite_rect.material();
                let mut data_ref = material.data_ref();

                // Load slime texture
                data_ref.bind("diffuseTexture", context.constructor.get_texture("data/characters/slime/slime-Sheet.png"));

                // Define idle animation
                self.animation = SpriteSheetAnimation::new("idle", 0.2, true);
                self.animation.add_frame_rect(Vector2::new(0.0, 0.0));
                self.animation.add_frame_rect(Vector2::new(0.25, 0.0));
                self.animation.add_frame_rect(Vector2::new(0.5, 0.0));
                self.animation.add_frame_rect(Vector2::new(0.75, 0.0));

                self.animation.restart();
            }
        }
    }

    fn on_update(&mut self, context: &mut ScriptContext) {
        self.animation.update(context.dt);

        if let Some(sprite_node) = context
            .scene
            .graph
            .try_get_mut(self.sprite)
            .and_then(|n| n.cast_mut::<Rectangle>()) {
            let rect = self.animation.current_frame_uv_rect().unwrap_or_default();
            sprite_node.set_uv_rect(rect);
        }
    }

    fn on_contact(
        &mut self,
        context: &mut ScriptContext,
        _this: Handle<Node>,
        contact_data: &ContactData,
    ) {
        for manifold in &contact_data.manifolds {
            for point in &manifold.points {
                if point.local_n1.y < 0.0 {
                    // Player jumped on us from above
                    context.message_sender.send(ScriptMessage {
                        receiver: context.handle,
                        message: EnemyMessage::PlayerLanded,
                    });
                }
            }
        }
    }
}
```

### ⚙️ Register Enemy Script

Inside your plugin implementation (`impl Plugin for PlatformerPlugin`), add:

```rust
script_constructors.add::<Enemy>("Enemy");
```

---

## 💥 Step 3: Handle Enemy Contact in Player Script

### ✅ Define Message Enum

Add this enum:

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
enum EnemyMessage {
    PlayerLanded,
}
```

### 🔄 Update Player Script to Handle Death

Modify `on_contact()` in `Player` script:

```rust
fn on_contact(
    &mut self,
    context: &mut ScriptContext,
    _this: Handle<Node>,
    contact_data: &ContactData,
) {
    let bodies = &context.scene.physics2d.bodies;

    for manifold in &contact_data.manifolds {
        for point in &manifold.points {
            if point.local_n1.y < 0.0 {
                self.is_grounded = true;
            } else if point.local_n1.y > 0.7 {
                // Player hit the bottom of the enemy — kill player
                self.die(context);
            }
        }
    }

    if self.is_grounded && contact_data.active_contacts > 0 {
        self.can_jump = true;
    }
}

fn die(&mut self, context: &mut ScriptContext) {
    // Disable rigid body or remove node
    if let Some(rigid_body) = context.scene.graph[context.handle].cast_mut::<RigidBody>() {
        rigid_body.set_lin_vel(Vector2::new(0.0, 0.0));
    }

    // TODO: Play death animation or remove from scene
    println!("Player died!");
}
```

---

## 🤖 Step 4: Add Basic Enemy AI (Movement)

### ✅ Make Enemy Dynamic

Back in the editor:
- Select enemy rigid body
- Change `Body Type` to **Dynamic**

### 🔄 Update Enemy Script for Movement

Add fields to `Enemy` struct:

```rust
#[derive(Visit, Reflect, Debug, Clone, Default, TypeUuidProvider)]
#[type_uuid(id = "a1b2c3d4-e5f6-7890-g1h2-i3j4k5l6m7n8")]
#[visit(optional)]
struct Enemy {
    sprite: Handle<Node>,
    animation: SpriteSheetAnimation,
    direction: f32,
    move_speed: f32,
}
```

Update `on_start()`:

```rust
fn on_start(&mut self, context: &mut ScriptContext) {
    self.direction = -1.0; // Start moving left
    self.move_speed = 1.0;

    // Animation setup as before...
}
```

Update `on_update()`:

```rust
fn on_update(&mut self, context: &mut ScriptContext) {
    if let Some(rigid_body) = context.scene.graph[context.handle].cast_mut::<RigidBody>() {
        let velocity = rigid_body.lin_vel();
        rigid_body.set_lin_vel(Vector2::new(self.direction * self.move_speed, velocity.y));
    }

    // Animation update as before...
}
```

---

## 🧪 Step 5: Test It!

Hit **Play** in the editor and test:
- Walk into enemy from side → should die
- Jump on enemy from above → enemy dies
- Enemy moves back and forth
- Animations play correctly

---

## 📝 Summary of What We Did in Part 4

| Feature | Description |
|--------|-------------|
| Enemy Entity | Created enemy with physics and sprite |
| Collision Handling | Detected player-enemy collisions |
| Player Death | Implemented basic death logic |
| Enemy AI | Added simple back-and-forth movement |
| Messages | Used `ScriptMessage` to communicate between scripts |

---

Continue with **Part 5**, where we add:
- **Level transitions**
- **Checkpoints**
- **Respawn system**
- **UI elements (health, score)**
