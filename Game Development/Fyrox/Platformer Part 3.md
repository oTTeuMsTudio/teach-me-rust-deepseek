Let's dive into **Part 3** of the Fyrox 2D Platformer Tutorial: [Fyrox Book - Platformer Part 3](https://fyrox-book.github.io/tutorials/platformer/part3.html)

---

# 🎮 Fyrox 2D Platformer - Part 3: Player Animation States

In this part, we will:
- Add **jump animation**
- Improve the **animation state machine**
- Use **sprite sheet frames** for idle, run, and jump
- Trigger animations based on player **state**

Let’s continue from where we left off in `game/src/lib.rs`.

---

## 🧠 Step 1: Update Player Struct with Animation Definitions

### ✅ Add Animation State Enum
Add this enum above your `Player` struct:

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
enum PlayerAnimationState {
    Idle,
    Run,
    Jump,
}
```

### 🔁 Update `Player` Struct
Replace the old `current_animation` field:

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
}
```

We now track the player’s **animation state**, not just an index.

---

## 🖼️ Step 2: Define Sprite Sheet Frame Regions

Add this helper function to define UV rectangles for each animation frame:

```rust
fn make_uv(x: f32, y: f32) -> Vector2<f32> {
    let w = 0.143;
    let h = 0.091;
    Vector2::new(x * w, y * h)
}
```

Now define animation frames inside `on_start()`:

```rust
fn on_start(&mut self, context: &mut ScriptContext) {
    if let Some(sprite_rect) = context.scene.graph[self.sprite].cast_mut::<Rectangle>() {
        let material = sprite_rect.material();
        let mut data_ref = material.data_ref();

        // Define animations manually
        let mut idle = SpriteSheetAnimation::new("idle", 0.2, true);
        idle.add_frame_rect(make_uv(0.0, 0.0));
        idle.add_frame_rect(make_uv(1.0, 0.0));
        idle.add_frame_rect(make_uv(2.0, 0.0));
        idle.add_frame_rect(make_uv(3.0, 0.0));

        let mut run = SpriteSheetAnimation::new("run", 0.1, true);
        run.add_frame_rect(make_uv(0.0, 1.0));
        run.add_frame_rect(make_uv(1.0, 1.0));
        run.add_frame_rect(make_uv(2.0, 1.0));
        run.add_frame_rect(make_uv(3.0, 1.0));
        run.add_frame_rect(make_uv(4.0, 1.0));
        run.add_frame_rect(make_uv(5.0, 1.0));

        let mut jump = SpriteSheetAnimation::new("jump", 0.1, false);
        jump.add_frame_rect(make_uv(0.0, 2.0));

        data_ref.bind("diffuseTexture", context.constructor.get_texture("data/characters/adventurer/adventurer-Sheet.png"));

        self.animations = vec![idle, run, jump];
        self.current_state = PlayerAnimationState::Idle;
    }
}
```

This replaces the previous `from_definition()` method with explicit frame definitions using our `make_uv()` helper.

---

## 🔄 Step 3: Update Animation Based on Player State

Update the `on_update()` method to change animation states:

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

        velocity.x = x_speed;

        if self.jump && self.is_grounded && self.can_jump {
            velocity.y = 5.0;
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

        // Determine new animation state
        let new_state = if !self.is_grounded {
            PlayerAnimationState::Jump
        } else if x_speed.abs() > 0.1 {
            PlayerAnimationState::Run
        } else {
            PlayerAnimationState::Idle
        };

        if new_state != self.current_state {
            self.current_state = new_state;

            // Restart animation when state changes
            match self.current_state {
                PlayerAnimationState::Idle => self.animations[0].restart(),
                PlayerAnimationState::Run => self.animations[1].restart(),
                PlayerAnimationState::Jump => self.animations[2].restart(),
            }
        }

        // Update active animation
        let anim_index = match self.current_state {
            PlayerAnimationState::Idle => 0,
            PlayerAnimationState::Run => 1,
            PlayerAnimationState::Jump => 2,
        };

        self.animations[anim_index].update(context.dt);

        if let Some(sprite) = context
            .scene
            .graph
            .try_get_mut(self.sprite)
            .and_then(|n| n.cast_mut::<Rectangle>()) {
            let rect = self.animations[anim_index].current_frame_uv_rect().unwrap_or_default();
            sprite.set_uv_rect(rect);
        }
    }
}
```

---

## 📂 Step 4: Assign Texture in Editor (Optional)

If you're using custom animation definitions, make sure the texture is assigned correctly in the editor:

1. Select the **player sprite node**.
2. In Inspector:
   - Open `Material`
   - Drag and drop `data/characters/adventurer/adventurer-Sheet.png` onto the `diffuseTexture` slot.

---

## 🧪 Step 5: Test It!

Hit **Play** in the editor and test:
- Stand still → should play **idle animation**
- Move left/right → should play **running animation**
- Jump → should play **jump animation once per jump**

---

## 📝 Summary of What We Did in Part 3

| Feature | Description |
|--------|-------------|
| Animation States | Added `Idle`, `Run`, and `Jump` states |
| Manual Animation Frames | Defined specific UV rectangles for each frame |
| Better Animation Control | Switched animations based on movement/jump state |
| Smooth Transitions | Restart animations when switching states |

---

Continue with **Part 4**, where we add:
- Enemy AI
- Collision detection
- Health system
- Damage and death logic
