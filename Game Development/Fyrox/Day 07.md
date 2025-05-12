# Building a full 3D game, add collectible items next — like coins or power-ups.

---

## 💰 Step: Add Collectible Items (e.g., Coins)

We'll:
1. Create floating coin models in the scene
2. Detect when the player touches them
3. Remove the coin and increase score/health

This is great for:
- Platformers
- Exploration games
- RPGs with pickups

---

### 📁 Folder Structure Reminder

Make sure your project looks like this:

```
your_project/
├── Cargo.toml
├── src/
│   └── main.rs
└── data/
    ├── models/
    │   ├── robot.glb     <-- your character
    │   └── coin.glb      <-- new coin model
    └── textures/
        └── ground.png
```

> You can find free `.glb` coin models on [Sketchfab](https://sketchfab.com) or [Google Poly](https://poly.google.com).

---

## ✅ Step 1: Load Coin Model in `init()`

Add this inside your `init()` method:

```rust
let resource_manager = context.resource_manager.clone();
let coin_model_future = resource_manager.request_model("data/models/coin.glb");

coin_model_future.then(|result| {
    if let Ok(model) = result {
        // Spawn multiple coins at different positions
        for i in 0..5 {
            let instance = context.scene.instantiate_model(&model);
            context.scene.graph[instance]
                .local_transform_mut()
                .set_position(Vector3::new(i as f32 * 3.0, 2.0, 0.0));
                
            self.coins.push(instance); // store handle to check collision later
        }
    }
});
```

> Don’t forget to add `coins: Vec<Handle<Node>>` to your `Game` struct!

Update your `Game` struct like this:

```rust
struct Game {
    player: Handle<Node>,
    camera: Handle<Node>,
    mouse_locked: bool,
    last_mouse_pos: (f64, f64),
    health_text: Handle<UiNode>,
    health: f32,
    coins: Vec<Handle<Node>>, // 👈 Add this line
}
```

Initialize it in `main()`:

```rust
engine.add_plugin(Game {
    ...
    coins: Vec::new(), // 👈 Initialize
});
```

---

## 🧠 Step 2: Check Collision Between Player & Coin

Inside your `update()` method:

```rust
let player_pos = context.scene.graph[self.player].global_position();

for coin in &self.coins {
    let coin_pos = context.scene.graph[*coin].global_position();
    let distance = (player_pos - coin_pos).norm();

    if distance < 1.0 { // adjust based on model size
        // Remove coin from scene
        context.scene.graph.remove(*coin);

        // Increase health/score
        self.health += 10.0;

        // Update UI text
        context.engine.get_ui().send_message(TextMessage::text(
            self.health_text,
            fyrox_ui::message::MessageDirection::ToWidget,
            format!("Health: {:.0}", self.health),
        ));
    }
}
```

---

## 🎉 Final Result

Now your game has:
✅ A 3D character that moves and jumps  
✅ Health bar that updates  
✅ Floating coins that disappear when collected  
✅ Score or health increases when collected  

---

## 🚀 Next:
1. 🎵 Add background music or sound effects (like coin pickup sound)
2. 🧟 Add enemies that follow the player
3. 🧱 Add shooting or attacking mechanics
4. 🏁 Add a win condition or level finish
