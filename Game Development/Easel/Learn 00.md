Learn and use [Easel](https://easel.games/), the 2D game programming language designed for making multiplayer games easily. This guide assumes no prior experience and will take you from setup to building your first game.

---

## 🧩 Step 1: Understand What Easel Is

### What is Easel?
- **Easel** is a **2D game programming language** that makes creating **multiplayer games** as easy as creating singleplayer ones.
- It handles all networking, synchronization, and rollback netcode automatically.
- Designed with **beginners** and **experts** in mind.
- Uses **Behaviors**, **Reactivity**, and **Hierarchy** to make code clean and readable.

---

## 🚀 Step 2: Get Started (No Setup Needed)

### Access the Editor
1. Go to: https://easel.games/
2. Click on **"Start Making Games"** or **"Editor"** (if available).
3. You’ll be taken to the **online game editor**.
   - No downloads or installations required.
   - Everything works in your browser!

---

## 📁 Step 3: Explore the Interface

Once inside the editor:

### Main Components:
1. **Project Explorer**: Lists all files and folders in your project.
2. **Code Editor**: Where you write your Easel code.
3. **Game Preview**: Shows what your game looks like while it runs.
4. **Toolbox / Palette**: Contains assets, components, and tools.

---

## 🧱 Step 4: Learn the Basic Concepts of Easel

Easel uses a unique approach to game development based on three pillars:

### 1. **Behaviors**
- Behaviors are sequences of steps your game objects can do.
- Think of them as "scripts" or functions that run over time.
- Example: “When this enemy sees the player, chase them.”

### 2. **Reactivity**
- Define how your game should react to changes.
- Use `on` statements to trigger actions when something happens.
- Example: `on collision with Player => destroy self`

### 3. **Hierarchy**
- Organize your game objects into parent-child relationships.
- Makes it easier to manage groups of objects (like a car with wheels, or a spaceship with turrets).

---

## 🎮 Step 5: Make Your First Game – A Multiplayer Bouncing Ball

Let’s create a simple game where players see a ball bounce around the screen — and everyone sees the same thing thanks to Easel's built-in multiplayer.

### Step 5.1: Create a New Project
1. In the editor, click **New Project**.
2. Name it something like **"Bouncing Ball"**.

### Step 5.2: Add a Ball Object
In the code editor, type:

```easel
object Ball {
  // Appearance
  shape = circle(20)
  color = red

  // Position
  x = 100
  y = 100

  // Velocity
  vx = 100
  vy = 100

  // Behavior
  behavior BounceAround {
    loop {
      move by (vx * dt, vy * dt)

      if (x < 0 || x > screen.width) {
        vx = -vx
      }

      if (y < 0 || y > screen.height) {
        vy = -vy
      }

      wait
    }
  }
}
```

### Step 5.3: Run the Game
1. Click the **Run** button.
2. You should see a red ball bouncing around the screen.

### Step 5.4: Play with Friends (Multiplayer!)
1. Click the **Share** button in the editor.
2. Copy the link and send it to a friend.
3. Both of you will see the same bouncing ball — in real-time!

> ✅ That’s multiplayer working automatically. You didn’t have to write any networking code!

---

## 🛠️ Step 6: Learn More Features

Now that you’ve made a basic game, let’s expand your knowledge.

### Add a Player-Controlled Character

```easel
object Player {
  shape = square(30)
  color = blue

  x = 200
  y = 200

  behavior MoveWithArrowKeys {
    loop {
      if (key.left.down) { x -= 200 * dt }
      if (key.right.down) { x += 200 * dt }
      if (key.up.down) { y -= 200 * dt }
      if (key.down.down) { y += 200 * dt }

      wait
    }
  }
}
```

This creates a blue square you can control with arrow keys — and again, this works across all players.

---

## 🧪 Step 7: Test and Debug

Use these tools to test your game:
- **Live Reloading**: Any change in code updates the running game instantly.
- **Print Statements**:
  ```easel
  print("Player x:", player.x)
  ```
- **Pause & Inspect**: Pause the game and inspect object properties.

---

## 💾 Step 8: Publish Your Game

Once your game is done:

1. Click **Publish** in the editor.
2. Choose a name and description.
3. Easel will host your game and give you a shareable link.
4. Anyone with the link can play your game — no download needed!

---

## 📚 Step 9: Keep Learning with Resources

### Official Resources:
- **Tutorials**: https://easel.games/learn
- **Documentation**: https://docs.easel.games/
- **Examples**: Look for sample projects in the editor (often labeled “Templates” or “Examples”).
- **Community Discord**: Join their community to ask questions and share ideas.

---

## 🏗️ Step 10: Advanced Tips (For Experts)

If you're experienced with coding:

- Use **external editors** like VS Code with Easel (via CLI tools).
- Build complex systems using **signals**, **components**, and **state machines**.
- Leverage **rollback netcode** for responsive gameplay.
- Customize **physics**, **UI**, **audio**, and more.

---

## ✅ Summary Checklist

| Step | Task |
|------|------|
| ✅ | Understand what Easel is |
| ✅ | Open the online editor |
| ✅ | Learn Behaviors, Reactivity, Hierarchy |
| ✅ | Create a bouncing ball |
| ✅ | Add a player-controlled character |
| ✅ | Share your game with friends |
| ✅ | Debug and publish |
| ✅ | Explore tutorials and docs |

---
