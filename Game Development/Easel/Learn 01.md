Awesome! Let's go a step further and build a **simple multiplayer platformer** using Easel. Since Easel is built for **real-time multiplayer games**, this will be a great way to explore what the platform can really do.

---

## 🕹️ Build a Multiplayer Platformer in Easel

We’ll create a basic 2D platformer where players can move left/right, jump, and see each other in real time.

---

### 🧰 Step 1: Open the Editor

Go to: [https://easel.games/editor](https://easel.games/editor)

You should see the editor with a default sprite (like a cat or square).

---

### 🧱 Step 2: Set Up the Player Sprite

Let’s rename the default sprite to “Player”.

#### Rename Your Sprite:
- In the top-left corner of the editor, click on the sprite name.
- Change it to `Player`.

---

### 🚦 Step 3: Add Controls

We want the player to be able to move left/right and jump.

#### Drag These Blocks Into the Workspace:

```plaintext
When Game Starts
    set gravity to 1000
```

From `Controls` category:

```plaintext
When left arrow key is pressed
    set x speed to -200

When right arrow key is pressed
    set x speed to 200

When up arrow key is pressed and Player is on ground
    set y speed to -500
```

> 💡 This lets your character move and jump only when touching the ground.

---

### 🧱 Step 4: Add Platforms

Click on the **Sprites** tab in the toolbox.

Drag out **“Add new sprite”** and name it `Platform`.

Change its shape to a rectangle (use the Shape tool), then resize and position it below the player so they land on it.

Duplicate the platform by right-clicking and selecting **"Duplicate"**, then spread them out to make a level.

---

### 🌐 Step 5: Enable Multiplayer

Easel makes this easy — just add one block!

From the `Multiplayer` category:

```plaintext
When Game Starts
    enable multiplayer
```

That’s it! Now multiple people can join the same game in real time.

Each player will control their own character and see others moving live.

---

### 🎮 Step 6: Test the Game

Click **▶ Run**.

Open another browser window or ask a friend to open the same link — you'll both see each other’s characters moving in real time!

---

### ☁️ Step 7: Publish & Share

Click the **Save** button and give your game a name like “My First Platformer”.

Then click **Publish**.

Easel gives you a shareable link that anyone can use to play your game — no download required!

---

## 🧠 Optional Enhancements

Want to make your platformer more fun? Try these ideas:

| Feature | How to Add It |
|--------|----------------|
| Score counter | Use Variables to count coins collected |
| Coins or collectibles | Add sprites and detect collision with Player |
| Health system | Use Variables and show hearts or numbers |
| Power-ups | Add special effects when Player touches items |
| Timer or levels | Use Scenes or countdown variables |

---

## 📽️ Want a Video Walkthrough?

If you'd prefer a video-style tutorial, I recommend checking out official Easel demos or tutorials on YouTube (search "Easel Games tutorial"). But if you’d like, I can guide you through building a **shooter**, **puzzle game**, or even a **multiplayer racing game** next!

---

Next:
- A **top-down shooter**
- A **racing game**
- A **chat-based multiplayer game**
