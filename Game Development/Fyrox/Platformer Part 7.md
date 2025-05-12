Let's continue with **Part 7** of the Fyrox 2D Platformer Tutorial: [Fyrox Book - Platformer Part 7](https://fyrox-book.github.io/tutorials/platformer/part7.html)

---

# 🔊 Fyrox 2D Platformer - Part 7: Adding Sound Effects and Music

In this part, we’ll:
- Add **background music**
- Play **sound effects** (jump, coin collect, death)
- Use **audio sources** in the scene
- Trigger sounds from script events

Let’s go step by step.

---

## 🎵 Step 1: Prepare Audio Assets

### ✅ Download or Add Sounds
Place these audio files in your `data/sounds/` folder:
- `music.ogg` – background music
- `jump.wav`
- `coin.wav`
- `death.wav`

> You can use free assets from sites like [freesound.org](https://freesound.org) or the tutorial's asset pack.

---

## 🔊 Step 2: Add Audio Listener and Sources

### ✅ Add an Audio Listener
This allows us to hear sound.

In the editor:
- Select the **player camera node**
- In Inspector → click `+ Component` → Add `Audio Listener`

### ✅ Add Audio Source for Background Music
1. Right-click `__ROOT__` → `Add Child → Audio Source`
2. Name it `BackgroundMusic`
3. In Inspector:
   - Load `data/sounds/music.ogg`
   - Set `Looped = true`
   - Set `Play Once = false`
   - Enable `Play On Start`

---

## 🧠 Step 3: Create Sound Player Script Helper

We'll create a helper function to play sounds from scripts.

### ✅ Add This Function to `lib.rs`

```rust
fn play_sound(context: &mut ScriptContext, sound_path: &str) {
    if let Some(sound) = context.constructor.get_sound(sound_path) {
        context.scene.sound_context.play(sound);
    }
}
```

---

## 💥 Step 4: Play Sound Effects from Scripts

### ✅ 1. Jump Sound

Inside `on_update()` in `Player` script:

```rust
if self.jump && self.is_grounded && self.can_jump {
    velocity.y = 5.0;
    self.can_jump = false;
    self.is_grounded = false;

    play_sound(context, "data/sounds/jump.wav");
}
```

---

### ✅ 2. Coin Collect Sound

Inside `on_message()` in `Player` script:

```rust
fn on_message(&mut self, context: &mut ScriptContext, message: &[u8]) {
    // Existing messages...

    if let Ok(msg) = bincode::deserialize::<CoinMessage>(message) {
        match msg {
            CoinMessage::Collected(_) => {
                self.score += 1;
                update_ui(context, self.score_text, format!("Score: {}", self.score));
                
                play_sound(context, "data/sounds/coin.wav");
            }
        }
    }

    // Other handlers...
}
```

---

### ✅ 3. Death Sound

Inside `die()` method in `Player` script:

```rust
fn die(&mut self, context: &mut ScriptContext) {
    self.health -= 1;

    play_sound(context, "data/sounds/death.wav");

    // Respawn logic as before...
}
```

---

## 🎮 Step 5: Optional – Enemy Death Sound

Update enemy collision handler:

```rust
fn on_contact(
    &mut self,
    context: &mut ScriptContext,
    _this: Handle<Node>,
    contact_data: &ContactData,
) {
    for manifold in &contact_data.manifolds {
        for point in &manifold.points {
            if point.local_n1.y < 0.0 {
                context.message_sender.send(ScriptMessage {
                    receiver: context.handle,
                    message: EnemyMessage::PlayerLanded,
                });

                play_sound(context, "data/sounds/enemy_death.wav");
            }
        }
    }

    // Other checks...
}
```

> Make sure you have `enemy_death.wav` in your `sounds` folder.

---

## 🎶 Step 6: Control Volume and Pause Music

Optionally, you can control volume and pause music via code.

### ✅ Add These Helper Functions

```rust
fn set_music_volume(context: &mut ScriptContext, volume: f32) {
    if let Some(music_handle) = context.scene.graph.find_by_name_from_root("BackgroundMusic").map(|(_, h)| h) {
        if let Some(source) = context.scene.graph[music_handle].cast_mut::<AudioSource>() {
            source.set_gain(volume);
        }
    }
}

fn stop_music(context: &mut ScriptContext) {
    if let Some(music_handle) = context.scene.graph.find_by_name_from_root("BackgroundMusic").map(|(_, h)| h) {
        if let Some(source) = context.scene.graph[music_handle].cast_mut::<AudioSource>() {
            source.stop();
        }
    }
}
```

Call them from input or UI buttons:
```rust
KeyCode::KeyM => {
    set_music_volume(context, 0.0); // Mute
}
```

---

## 🧪 Step 7: Test It!

Hit **Play** and test:
- Press Space → hear jump sound
- Collect coin → hear coin sound
- Die → hear death sound
- Music plays continuously in the background

---

## 📝 Summary of What We Did in Part 7

| Feature | Description |
|--------|-------------|
| Audio Listener | Added so player can hear sound |
| Background Music | Looping music using `AudioSource` |
| Sound Effects | Played jump, coin, death sounds |
| Script Integration | Used `play_sound()` helper |
| Volume Control | Optional volume/mute functions |

---

Continue with **Part 8**, where we add:
- **Polish features**
- **Particles (dust jumps, coin sparks)**
- **Better animations**
- **Visual polish**
