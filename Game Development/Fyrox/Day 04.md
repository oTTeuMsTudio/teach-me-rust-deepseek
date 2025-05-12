# You're all in — let's keep building your 3D game step-by-step.

Since you've already got movement and camera control working, the next logical step is:

---

## 🌍 1. Add Ground (and Stop Falling Through the Floor!)

Let’s add a ground plane so your player doesn’t fall forever.

### ✅ Code to Add Ground

Inside the `init()` method of your `Game` plugin, add this **after** creating the camera and player:

```rust
// Create ground
let ground = MeshBuilder::new(
    BaseBuilder::new()
        .with_local_transform(
            TransformBuilder::new()
                .with_local_position(Vector3::new(0.0, -1.0, 0.0)) // slightly below player
                .build()
        ),
    CubeBuilder::new()
        .with_size(100.0)
        .build(context.geometry_cache.clone()),
)
.with_diffuse_texture(Some("data/textures/ground.png".into()))
.build(context.scene);

// Make it static (so physics doesn't move it)
context.scene.graph[ground]
    .as_rigid_body_mut()
    .set_body_type(rapier3d::prelude::RigidBodyType::Static);
```

> 💡 Tip: You can use any large cube or plane as ground. Also, make sure you have a `ground.png` texture in `data/textures/`.

---

## ⬇️ 2. Add Gravity

Fyrox has a built-in physics engine. Let’s enable gravity so the player falls onto the ground.

### 🔧 Enable Gravity in `main.rs`

In your `main()` function, before running the event loop, enable gravity:

```rust
engine.physics_world_mut().gravity = Vector3::new(0.0, -9.81, 0.0);
```

Now your player will fall down and hit the ground!

---

## 🕹️ 3. Add Jumping

We’ll let the player press **Spacebar** to jump.

### ✅ Add Jump Logic in `update()`

Inside the `update()` function:

```rust
if input.key_is_down(VirtualKeyCode::Space) {
    let mut body = context.scene.graph[self.player].as_rigid_body_mut();
    if body.is_on_ground() { // only jump if on ground
        body.set_lin_vel(Vector3::new(0.0, 5.0, 0.0)); // jump up
    }
}
```

> Note: `is_on_ground()` works if you have proper colliders set up.

---

## 🧱 Final Result

You now have:
✅ Movement with WASD  
✅ Mouse look for camera  
✅ Ground so you don’t fall forever  
✅ Gravity pulling the player down  
✅ Jumping with Spacebar  

---

## 🚀 Next:
- 👤 Replace capsule with a 3D character model
- 🎵 Add background music or sound effects
- 🖼️ Add a UI health bar that decreases when falling
- 🌟 Add collectible items (like coins or power-ups)
