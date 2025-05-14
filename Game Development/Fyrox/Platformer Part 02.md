Let's continue with **Part 2** of the Fyrox 2D Platformer Tutorial, following [Fyrox Book - Platformer Part 2](https://fyrox-book.github.io/tutorials/platformer/part2.html).

This part focuses on:
- Adding **gravity**
- Implementing **ground detection**
- Handling **jump states**
- Improving **player physics**

---

# 🧱 Fyrox 2D Platformer - Part 2: Gravity and Jump Mechanics

---

## 🔁 Step 1: Update Player Script for Ground Detection

### ✅ Add New Fields to `Player` Struct
Update your `Player` struct in `game/src/lib.rs`:

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
    is_grounded: bool,
    can_jump: bool,
}
```

We added:
- `is_grounded`: whether the player is touching the ground.
- `can_jump`: prevents double-jumping.

---

## 🧠 Step 2: Enable Physics Contact Events

To detect when the player touches the ground, we need to listen to contact events.

### ✅ Enable Contact Events in RigidBody Settings

In the editor:
1. Select the **player's rigid body node**.
2. In Inspector:
   - Go to `Rigid Body > Additional Properties`
   - Check `Enable Contact Events`

This allows us to receive collision events in code.

---

## 🔄 Step 3: Override `on_contact()` in Player Script

Add this method to your `impl ScriptTrait for Player`:

```rust
fn on_contact(
    &mut self,
    context: &mut ScriptContext,
    _this: Handle<Node>,
    contact_data: &ContactData,
) {
    // Only react if at least one collider belongs to the player
    let bodies = &context.scene.physics2d.bodies;
    let body = bodies.get(contact_data.rigid_body_pair.0).unwrap();
    let other_body = bodies.get(contact_data.rigid_body_pair.1).unwrap();

    // Check if any contact point is below the player
    for manifold in &contact_data.manifolds {
        for point in &manifold.points {
            if point.local_n1.y < 0.0 {
                self.is_grounded = true;
                break;
            }
        }
    }

    // Allow jumping again after landing
    if self.is_grounded && contact_data.active_contacts > 0 {
        self.can_jump = true;
    }
}
```

This checks if the player is standing on something by looking at the **normal vector** of the contact points.

---

## 🕹️ Step 4: Improve Movement and Jump Logic

Replace the previous movement logic inside `on_update()` with this improved version:

```rust
fn on_update(&mut self, context: &mut ScriptContext) {
    if let Some(rigid_body) = context.scene.graph[context.handle].cast_mut::<RigidBody>() {
        let x_speed = if self.move_left {
            -3.0
        } else if self.move_right {
            3.0
        } else {
            0.0
        };

        let mut velocity = rigid_body.lin_vel().clone();

        // Apply horizontal movement
        velocity.x = x_speed;

        // Apply jump
        if self.jump && self.is_grounded && self.can_jump {
            velocity.y = 5.0; // Jump strength
            self.can_jump = false;
            self.is_grounded = false;
        }

        rigid_body.set_lin_vel(velocity);

        // Reset grounded flag if not touching anything
        if !self.is_grounded {
            context.query_context.physics2d_context.contact_events.clear();
        }

        // Flip sprite based on direction
        if x_speed != 0.0 && let Some(sprite_node) = context.scene.graph.try_get_mut(self.sprite) {
            let transform = sprite_node.local_transform_mut();
            let scale = **transform.scale();
            if (x_speed < 0.0 && scale.x > 0.0) || (x_speed > 0.0 && scale.x < 0.0) {
                transform.set_scale(Vector3::new(-scale.x, scale.y, scale.z));
            }
        }

        // Animation logic here...
    }
}
```

> ✅ This ensures that:
> - The player only jumps once per ground contact.
> - Gravity pulls the player down naturally (via engine physics).
> - Sprite flips correctly when changing direction.

---

## 🌍 Step 5: Tune Physics World Gravity

By default, gravity may be too weak or strong. You can adjust it in the scene settings.

### 🛠️ Set Global Gravity in Editor:
1. Open the `Scene Settings` panel.
2. Find `Physics2D → Gravity`.
3. Set to `(0.0, -9.8)` for Earth-like gravity.

You can tweak the Y value later for better gameplay feel.

---

## 🚦 Step 6: Optional - Visualize Contact Points (Debug)

If you're having trouble detecting ground, enable debug drawing:

### 💡 In `Plugin` implementation (`lib.rs`):
Inside `on_register`, enable debug rendering:

```rust
context.renderer.enable_debug_renderer(true);
```

Now you'll see:
- Collision shapes
- Contact normals
- Velocity vectors

Useful for debugging why the player isn't detecting ground!

---

## 🧪 Step 7: Test Again

Hit **Play** in the editor and test:
- Walk left/right
- Press space while on the ground — should jump
- Should not double-jump
- Should land and reset jump ability

---

## 📝 Summary of What We Did in Part 2

| Feature | Description |
|--------|-------------|
| Ground Detection | Used contact events to determine if player is on the ground |
| Jump Logic | Prevents double-jumps, resets on landing |
| Improved Movement | More natural-feeling motion |
| Gravity | Applied via global physics world |
| Animation Prep | Ready for idle/run/jump animation states |

---

Continue with **Part 3**, where we add:
- Jump animation
- Running animation
- Better state machine for player actions?
