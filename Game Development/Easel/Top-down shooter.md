Let’s build a **simple top-down shooter** in Easel. This will be a 2D game where players can move around with arrow keys or WASD, shoot bullets with the spacebar or mouse, and enemies spawn to chase them.

Since Easel supports real-time multiplayer out of the box, this game could easily be expanded for co-op or competitive play later on!

---

## 🎯 Top-Down Shooter – Step-by-Step

### 🧰 Step 1: Open the Editor

Go to: [https://easel.games/editor](https://easel.games/editor)

You’ll see a default sprite (like a cat or square). We’ll turn this into your **player character**.

---

### 👨‍🚀 Step 2: Rename Your Player Sprite

Click on the sprite name at the top-left corner of the editor and rename it to `Player`.

---

### 🚦 Step 3: Add Movement Controls

We want the player to move in all directions using arrow keys or WASD.

#### From the `Controls` category:

```plaintext
When Game Starts
    set rotation style to "Free"

When left arrow key is pressed
    change x by -5

When right arrow key is pressed
    change x by 5

When up arrow key is pressed
    change y by -5

When down arrow key is pressed
    change y by 5
```

> 💡 You can also use `W`, `A`, `S`, `D` keys if preferred — just click the dropdown in each block.

---

### 🔫 Step 4: Add Shooting Mechanics

Let’s make the player shoot bullets when pressing the **spacebar**.

#### From `Controls`:

```plaintext
When space key is pressed
    create bullet at Player position
    point bullet in direction Player is facing
    set bullet speed to 10
```

> 💡 To make this work, you need to define what a **bullet** is.

---

### 💥 Step 5: Create the Bullet Sprite

From the `Sprites` category:

1. Drag **“Add new sprite”** and name it `Bullet`.
2. Change its shape to a small circle or rectangle.
3. Set the bullet to disappear after a short time:
   ```plaintext
   When Bullet starts
       wait 2 seconds
       delete this sprite
   ```

---

### 👹 Step 6: Add Enemies That Chase the Player

Let’s add simple enemy AI that spawns and follows the player.

#### A. Create Enemy Sprite

From `Sprites`, add a new sprite called `Enemy`.

Set its appearance to a red square or any shape you like.

#### B. Make Enemy Chase Player

In the Enemy script:

```plaintext
When Game Starts
    repeat forever
        face towards Player
        move forward 2
        wait 0.1 seconds
```

This makes enemies constantly chase the player.

---

### 🐭 Optional: Mouse Aim (Advanced)

If you want the player to aim toward the mouse cursor:

```plaintext
When Game Starts
    repeat forever
        point Player towards mouse
```

Now bullets will shoot in the direction the player is aiming.

---

### 💥 Step 7: Add Collision Detection

Make bullets damage enemies on contact.

In the **Bullet** script:

```plaintext
When Bullet hits Enemy
    delete this sprite
    delete Enemy
```

This deletes both the bullet and the enemy when they collide.

---

### 🧱 Step 8: Add Boundaries (Optional)

To prevent the player from moving off-screen:

```plaintext
When Game Starts
    repeat forever
        if Player x < -240 then set x to -240
        if Player x > 240 then set x to 240
        if Player y < -180 then set y to -180
        if Player y > 180 then set y to 180
        wait 0.1 seconds
```

You can adjust the numbers based on your screen size.

---

### 🕐 Step 9: Spawn Enemies Over Time

Let’s make enemies spawn every few seconds.

Create a new script (use the `Events` category):

```plaintext
When Game Starts
    repeat forever
        wait 3 seconds
        create Enemy at random position
```

Now enemies will keep spawning as the game goes on.

---

### 🌐 Step 10: Enable Multiplayer (Optional but Fun!)

Just like before, enable multiplayer support with one block:

```plaintext
When Game Starts
    enable multiplayer
```

Now multiple players can join and play together — shoot enemies and see each other in real time!

---

### ☁️ Step 11: Save & Publish

Once you're happy with how the game works:

1. Click the **Save** button (floppy disk icon).
2. Give your game a name like “Top-Down Shooter”.
3. Click **Publish** (cloud icon).
4. Copy the link and share it with friends or embed it anywhere!

---

## 🧠 Ideas to Expand the Game

Here are some ideas to improve your top-down shooter:

| Feature | How to Add It |
|--------|----------------|
| Score counter | Use Variables to count destroyed enemies |
| Health system | Add lives or health bar to Player |
| Power-ups | Create items that boost speed, fire rate, etc. |
| Boss enemies | Add stronger enemies with unique behavior |
| Level progression | Increase difficulty over time or unlock levels |

---

Next: **score tracking**, **health**, or making this a **co-op shooter**

Awesome! Let’s take your **top-down shooter** and turn it into a **real-time co-op shooter**, where multiple players can join, move independently, shoot enemies, and survive together.

Since Easel supports real-time multiplayer natively, we just need to tweak the game slightly to make sure all players are synced and working as a team.

---

# 🧑‍🤝‍🧑 Step-by-Step: Make Your Shooter a **Co-op Game**

We’ll build off your existing top-down shooter and add features like:

- Multiple player characters
- Shared enemy spawns (visible to everyone)
- Optional score tracking for teamwork

---

## ✅ Prerequisites

Make sure you’ve completed:
- The basic **top-down shooter** (movement, shooting, enemies)
- Basic **multiplayer setup** (`enable multiplayer` block)

If not, go back to the previous steps or let me know and I’ll recap!

---

## 🎮 Step 1: Allow Multiple Players to Join

Easel handles this automatically once you enable multiplayer.

### In the Player Sprite:

```plaintext
When Game Starts
    enable multiplayer
```

This will allow multiple players to connect and control their own `Player` sprite.

> 💡 Each connected player gets their own instance of the `Player` sprite. This is perfect for co-op!

---

## 🌐 Step 2: Make Enemies Visible to All Players

By default, some sprites may only appear on one player's screen. We want enemies to be visible to everyone.

### In the Enemy Sprite:

```plaintext
When Game Starts
    set global visibility
```

This ensures that **all players see the same enemies**.

You should also make sure **bullets** are visible to everyone:

### In the Bullet Sprite:

```plaintext
When Bullet starts
    set global visibility
```

Now bullets and enemies will appear across all player sessions.

---

## 🧠 Step 3: (Optional) Add Team Score or Health

Let’s create a **shared score variable** so all players can see how many enemies they've destroyed together.

### Create a Global Score Variable

From the `Variables` category:

```plaintext
When Game Starts
    create global variable "score"
    set score to 0
    show score
```

### Update the Score When an Enemy Dies

In the **Bullet** script:

```plaintext
When Bullet hits Enemy
    change score by 1
    delete Enemy
    delete Bullet
```

Now every time any player kills an enemy, the **team score increases**!

---

## 🛡️ Step 4: (Optional) Add Shared Health or Lives

To make it more co-op friendly, you can have a shared health bar or lives counter.

### Create a Global Health Variable

```plaintext
When Game Starts
    create global variable "health"
    set health to 10
    show health
```

### Reduce Health When Enemy Hits Player

Create a new script in the **Enemy** sprite:

```plaintext
When Enemy touches Player
    change health by -1
    delete this sprite
```

> 💡 You could also add a message like `"say 'Player hit!' for 1 sec"` to give feedback.

If health reaches zero, you can end the game for everyone.

---

## 🔁 Step 5: Improve Spawning So It Works Across Players

Make sure enemy spawning happens globally, not per-player.

Go to your **enemy spawner logic** (probably in a script outside any sprite):

```plaintext
When Game Starts
    repeat forever
        wait 3 seconds
        create Enemy at random position
        set Enemy global visibility
```

This ensures enemies spawn in the same world space for all players.

---

## 🧩 Step 6: Test the Co-op Game

Click **▶ Run**.

Open another browser window or share the link with a friend — both of you should see:

- Independent player movement
- Shared enemies
- Shared bullets
- Shared score/health

🎉 You now have a fully functional **co-op shooter** built in Easel!

---

## 📦 Bonus Features You Can Add

| Feature | How to Do It |
|--------|----------------|
| Power-ups | Add collectible sprites that boost fire rate or speed |
| Wave system | Increase number of enemies over time |
| Boss fights | Add special enemies with unique behavior |
| Chat messages | Use text input + display messages above players |
| Custom skins | Let players choose different colors or shapes |

---

## 🚀 Ready to Publish?

Once you're happy with your co-op shooter:

1. Click the **Save** button.
2. Give your game a name like “Co-op Shooter”.
3. Click **Publish**.
4. Share the link with friends or embed it anywhere!

---

Next: adding **chat**, **power-ups**, or turning this into a **zombie survival co-op game**
