Let’s get you started building your first game using **[Easel.games](https://easel.games/)**. Since Easel uses a **visual scripting system** (similar to Scratch or Blockly), it's beginner-friendly and doesn’t require you to write traditional code.

---

## 🎮 Step-by-Step: Create Your First Game on Easel

### 🔧 Step 1: Go to the Editor
1. Open your browser and go to [https://easel.games/editor](https://easel.games/editor)
2. You should see the **Easel editor interface** — no login required!

---

### 🧱 Step 2: Understand the Interface

Here’s what you’ll see in the editor:

- **Left Panel**: The **Toolbox** with blocks like:
  - `Sprites`
  - `Controls`
  - `Events`
  - `Variables`, etc.
- **Center Panel**: This is where you **build your scripts** by dragging and snapping blocks together.
- **Right Panel**: The **Game View**, showing how your game looks as it runs.

---

### 🚀 Step 3: Make a Simple Clicker Game

Let’s create a simple **clicker game** where clicking a sprite increases a score.

#### 👇 Part A: Add a Sprite

1. In the **Toolbox**, click on `Sprites`.
2. Drag the **“Add a new sprite”** block into the center panel.
3. A new sprite will appear in the game view (a colored square or circle).

> 💡 Tip: You can rename the sprite to something like “Button” for clarity.

---

#### 👇 Part B: Create a Score Variable

1. Click on the `Variables` category in the Toolbox.
2. Drag out the **“Create variable”** block.
3. Name the variable `score`.

Now, you have a variable called `score` that you can use in your game.

---

#### 👇 Part C: Make the Sprite Respond to Clicks

1. From the `Events` category, drag the **“When this sprite is clicked”** block.
2. Inside that block, do the following:
   - From `Variables`, add the **“Change score by 1”** block.
   - From `Sprites`, optionally add a **“Say ‘Clicked!’ for 1 sec”** block to give visual feedback.

Your script should look like this:

```plaintext
When [Button] is clicked
    change score by 1
    say "Clicked!" for 1 second
```

---

#### 👇 Part D: Display the Score

1. From `Variables`, drag the **“Show variable score”** block anywhere in the workspace.
2. The score will now appear on the screen when you run the game.

---

### ▶️ Step 4: Test Your Game

Click the **▶ Run** button at the top of the editor.

Try clicking on your sprite. The score should increase and show a message saying “Clicked!”

---

### ☁️ Step 5: Save & Publish Your Game

1. Click the **Save** icon (floppy disk) in the top menu.
2. Give your project a name like “My First Clicker Game”.
3. When you're ready, click **Publish** (cloud icon).
4. Easel will generate a **shareable link** you can send to friends or embed on a website.

---

### 📤 Bonus: Share It Online

Once published:
- Copy the link from the address bar.
- Share it on social media, Discord, or platforms like itch.io.

You can also:
- Embed it on a blog or portfolio site using the **Embed Code** option.
- Submit it to game jams or share it in the Easel Discord community (if available).

---

## 🧠 Want to Learn More?

Here are some next steps after your first clicker game:

| Goal | How to Do It |
|------|--------------|
| Add sound effects | Use the `Sound` category in the Toolbox |
| Add multiple sprites | Drag more sprites from the `Sprites` category |
| Make enemies move | Use `Motion` blocks and `Loops` |
| Build levels | Use `Variables` or `Scenes` if available |
| Make multiplayer games | Easel supports real-time multiplayer out-of-the-box! |

---

Next: a video tutorial-style walkthrough or try making a different type of game (like platformer, shooter, puzzle)
