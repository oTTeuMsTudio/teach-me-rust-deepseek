Let’s break down what [Easel](https://easel.games/) is and how you can start learning to use it.

---

## 🧠 What is Easel?

**Easel** is a **2D game programming language** designed to make creating games—especially **multiplayer games**—simple, fun, and accessible for both beginners and experienced developers. It abstracts away many of the complex parts of game development like networking, synchronization, and game loops so you can focus on building your game.

---

## 🎮 Key Features of Easel

### 1. **Effortless Multiplayer**
- You code as if all players are in the same world.
- Easel handles the hard stuff: **networking**, **synchronization**, and even **rollback netcode** (for smooth online play).
- No need to manage client-server architecture manually.

### 2. **Streamlined Programming Model**
- Uses a **hierarchical structure**, **reactive style**, and **behaviors** model to simplify game logic.
- Makes it easy to write clean, readable, and maintainable code.

### 3. **Beginner-Friendly**
- Designed to be simple enough for first-time coders.
- Fun and engaging because you're making games right from the start.

### 4. **All-in-One Game Development System**
- Includes tools for:
  - Graphics
  - Audio
  - Physics
  - UI
- Has an **online editor** so you can start coding instantly without installing anything.
- Games are hosted on Easel's servers and playable in any browser.

---

## 🧱 Core Concepts in Easel

### 1. **Behaviors**
Behaviors let you define step-by-step sequences that objects in your game follow. Think of them like scripts or AI routines that control how something acts over time.

**Example (pseudo-code):**
```easel
behavior PlayerMovement {
  every frame {
    if (keyPressed("left")) {
      player.x -= 5;
    }
    if (keyPressed("right")) {
      player.x += 5;
    }
  }
}
```

### 2. **Reactivity**
React to changes in your game using a declarative style. When something happens (like a collision), your code reacts automatically.

**Example (pseudo-code):**
```easel
when (player collides with enemy) {
  player.health -= 10;
  show "Game Over" if player.health <= 0;
}
```

### 3. **Hierarchy**
Organize your game into nested structures that mirror real-world relationships (e.g., a car has wheels, which have properties).

**Example (pseudo-code):**
```easel
object Car {
  object Wheel {
    property radius = 10;
    behavior Rotate {
      every frame { rotate by speed; }
    }
  }
}
```

---

## 🛠️ Getting Started with Easel

### Step 1: Go to the Editor
Visit: https://editor.easel.games/  
No installation needed—you can start coding right away in your browser!

### Step 2: Create Your First Game Object
Try creating a simple object and moving it around.

**Example: Moving a square**
```easel
object Player {
  shape = rectangle(20, 20);
  color = red;

  behavior Move {
    every frame {
      if (keyPressed("left")) x -= 5;
      if (keyPressed("right")) x += 5;
    }
  }
}
```

### Step 3: Add Multiplayer Support
To make your game multiplayer, just add:

```easel
multiplayer = true;
```

Now anyone can join your game through a link and interact with the shared world.

---

## 📚 Resources to Learn More

- **Official Website**: https://easel.games/
- **Online Editor**: https://editor.easel.games/
- **Community & Examples**: Join their Discord or look at example games in the editor.

---

## 💡 Tips for Learning Easel

- Start small: Make a single-player platformer or top-down shooter.
- Experiment with behaviors and reactivity.
- Try adding multiplayer support once you're comfortable.
- Read others’ code in the editor to see how they solve problems.
- Ask questions in the community—many people love helping new users!

---

## 🧩 Example: Simple Multiplayer Game

Here’s a basic multiplayer game where each player controls a colored square:

```easel
multiplayer = true;

object Player {
  shape = rectangle(30, 30);
  color = randomColor();

  behavior Movement {
    every frame {
      if (keyPressed("left")) x -= 5;
      if (keyPressed("right")) x += 5;
      if (keyPressed("up")) y -= 5;
      if (keyPressed("down")) y += 5;
    }
  }
}
```

This will create a unique player for each person who joins, each with their own color and movement.

---

Next: more specific tutorial or help with a particular type of game
