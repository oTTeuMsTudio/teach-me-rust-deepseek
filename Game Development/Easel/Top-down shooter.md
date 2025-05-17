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
