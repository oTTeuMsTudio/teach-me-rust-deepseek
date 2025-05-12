# Let’s keep going. Since you’re loving this, I’ll help you **add a 3D character model** to replace the capsule.

---

## 👤 Step: Replace Capsule with a 3D Character Model

We’ll:
1. Load a `.glb` or `.fbx` model (e.g., a robot or humanoid)
2. Position it where the player is
3. Make sure movement and camera still work with the new model

This will make your game look much more like a real game!

---

### 🔧 Folder Structure Reminder

```
your_project/
├── Cargo.toml
├── src/
│   └── main.rs
└── data/
    ├── models/
    │   └── robot.glb     <-- your 3D model here
    └── textures/
        └── ground.png    <-- optional
```

---

## ✅ Step 1: Add Model Loading Code

Replace the capsule creation code in `init()` with this:

```rust
// Load 3D model
let resource_manager = context.resource_manager.clone();
let model_future = resource_manager.request_model("data/models/robot.glb");

model_future.then(|result| {
    if let Ok(model) = result {
        // Spawn model at player position
        let instance_handle = context.scene.instantiate_model(&model);

        // Store handle to control position later
        self.player = instance_handle;
    }
});
```

> ⚠️ Note: This assumes your model has animations and is exported correctly. If not, we can fix that too!

---

## 🔁 Step 2: Adjust Camera Position

Your model might be taller than the capsule, so adjust the camera height:

```rust
// Set camera higher if using a humanoid model
self.camera = CameraBuilder::new(
    BaseBuilder::new()
        .with_local_transform(
            TransformBuilder::new()
                .with_local_position(Vector3::new(0.0, 1.8, -5.0)) // eye level for humanoid
                .build()
        )
).build(context.scene);
```

---

## 🎭 Step 3: Optional – Play Animation

If your model has animations (like idle or walk), you can play them:

```rust
if let Some(animation_controller) = context.scene.graph[self.player].as_model_mut().animation_controller_mut() {
    animation_controller.play("Idle", 1.0, true); // assumes "Idle" is animation name
}
```

> 💡 Tip: You can find free animated GLB models on [Sketchfab](https://sketchfab.com) or [Google Poly](https://poly.google.com).

---

## ✅ Final Result

Now you have:
- A full 3D character walking around
- Mouse look working
- Jumping and gravity
- Ground underfoot

It’s starting to feel like a real game! 🎮

---

## 🚀 Next:
1. 🎵 Add background music or jump sound
2. 🖼️ Add UI health bar or score counter
3. 💥 Add collectible items (coins, power-ups)
4. 🧟 Add enemies that follow or attack the player
