# Keep building your 3D game.

Since you've already got:
- Movement
- Camera
- Gravity
- Jumping
- A 3D character model
- UI (health bar)
- Collectible coins

Now let’s add **enemies that follow the player** to make things more exciting!

---

## 🧟 Step: Add Enemies That Follow the Player

We'll:
1. Load an enemy model (like a monster or robot)
2. Make it follow the player automatically
3. Optionally, reduce health when touching the enemy

This is perfect for:
- Survival games
- Horror games
- Action/platformers

---

### 📁 Folder Structure Reminder

Make sure you have this file:

```
data/models/monster.glb   <-- Your enemy model
```

> You can use any `.glb` or `.fbx` model — even a simple cube will work as a placeholder.

---

## ✅ Step 1: Load Enemy Model in `init()`

Inside your `init()` method:

```rust
let resource_manager = context.resource_manager.clone();
let enemy_model_future = resource_manager.request_model("data/models/monster.glb");

enemy_model_future.then(|result| {
    if let Ok(model) = result {
        // Spawn enemy at position
        let enemy = context.scene.instantiate_model(&model);
        context.scene.graph[enemy]
            .local_transform_mut()
            .set_position(Vector3::new(5.0, 1.0, 5.0)); // place enemy somewhere

        self.enemies.push(enemy); // store handle
    }
});
```

> Don’t forget to add `enemies: Vec<Handle<Node>>` to your `Game` struct!

Update your `Game` struct like this:

```rust
struct Game {
    player: Handle<Node>,
    camera: Handle<Node>,
    mouse_locked: bool,
    last_mouse_pos: (f64, f64),
    health_text: Handle<UiNode>,
    health: f32,
    coins: Vec<Handle<Node>>,
    enemies: Vec<Handle<Node>>, // 👈 Add this line
}
```

Initialize in `main()`:

```rust
engine.add_plugin(Game {
    ...
    enemies: Vec::new(), // 👈 Initialize
});
```

---

## 🧠 Step 2: Make Enemy Follow Player in `update()`

Add this inside your `update()` method:

```rust
let player_pos = context.scene.graph[self.player].global_position();

for enemy in &self.enemies {
    let enemy_pos = context.scene.graph[*enemy].global_position();
    
    // Move toward player
    let direction = (player_pos - enemy_pos).normalize();
    let speed = 0.05;

    context.scene.graph[*enemy]
        .as_rigid_body_mut()
        .set_lin_vel(direction * speed);

    // Optional: Hurt player on contact
    let distance = (player_pos - enemy_pos).norm();
    if distance < 0.75 {
        self.health -= 0.1; // slowly drain health

        context.engine.get_ui().send_message(TextMessage::text(
            self.health_text,
            fyrox_ui::message::MessageDirection::ToWidget,
            format!("Health: {:.0}", self.health),
        ));
    }
}
```

---

## 💥 Final Result

Now your game has:
✅ A 3D player with movement and jumping  
✅ Health system with UI  
✅ Coins to collect  
✅ Enemies that follow the player  
✅ Damage when near enemies  

---

## 🚀 Next:
1. 🔫 Add shooting or attacking mechanics
2. 🎵 Add background music or sound effects (coin pickup, damage)
3. 🏁 Add a win condition (e.g., collect all coins to finish level)
4. 🪄 Add power-ups (speed boost, invincibility)
