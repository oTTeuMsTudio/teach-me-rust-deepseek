# Add a simple UI next — like a health bar or score counter

---

## 🖼️ Step: Add a Simple UI (Text + Health Bar)

We'll:
1. Display text like "Health: 100"
2. Show a health bar that shrinks when the player falls
3. Keep everything working with Fyrox's built-in UI system

This is perfect for:
- Showing lives or score
- Making pause menus
- Creating HUDs and inventory screens

---

### 🔧 Step 1: Add `fyrox-ui` to `Cargo.toml`

```toml
[dependencies]
fyrox = "0.37"
fyrox-ui = "0.10" # or latest version
```

---

### ✅ Step 2: Create a Text Label in `init()`

Add this inside your plugin’s `init()` method:

```rust
// Create UI root
let mut ui_root = UiRoot::new();

// Add health text
let health_text = TextBuilder::new(fyrox_ui::widget::WidgetBuilder::new())
    .with_text("Health: 100")
    .build(&mut ui_root.ui.build_ctx());

// Store handle so we can change it later
self.health_text = health_text;
context.engine.set_ui(ui_root);
```

> ⚠️ Don’t forget to add `health_text: Handle<UiNode>` to your `Game` struct!

Update your `Game` struct like this:

```rust
struct Game {
    player: Handle<Node>,
    camera: Handle<Node>,
    mouse_locked: bool,
    last_mouse_pos: (f64, f64),
    health_text: Handle<UiNode>, // 👈 Add this line
    health: f32,
}
```

And initialize it in `main()`:

```rust
engine.add_plugin(Game {
    player: Default::default(),
    camera: Default::default(),
    mouse_locked: false,
    last_mouse_pos: (0.0, 0.0),
    health_text: Default::default(), // 👈 Initialize
    health: 100.0,
});
```

---

### 💥 Step 3: Hurt Player on Fall & Update UI

Inside `update()`, check if the player fell far and reduce health:

```rust
let position = context.scene.graph[self.player].global_position();
if position.y < -10.0 {
    self.health -= 10.0;
    
    // Reset player height
    context.scene.graph[self.player]
        .local_transform_mut()
        .set_position(Vector3::new(0.0, 5.0, 0.0));

    // Update UI text
    context.engine.get_ui().send_message(TextMessage::text(
        self.health_text,
        fyrox_ui::message::MessageDirection::ToWidget,
        format!("Health: {:.0}", self.health),
    ));
}

if self.health <= 0.0 {
    println!("Game Over!");
}
```

---

## 📊 Final Result

Now your game has:
✅ A 3D character model  
✅ Movement and jumping  
✅ Ground collision  
✅ A real-time health display  
✅ Player resets if they fall too far  

---

## 🚀 Next:
1. 🎵 Add background music or sound effects
2. 💰 Add collectible items (coins, power-ups)
3. 🧟 Add enemies that follow the player
4. 🧱 Add shooting or attacking
