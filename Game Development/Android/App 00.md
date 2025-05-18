# Create an Android game using **Easel** 

Easel is a powerful, beginner-friendly, 2D game programming language that makes it easy to build and deploy multiplayer games. While Easel doesn't explicitly mention "Android" in its documentation, the platform allows you to create browser-based games that can be played on any device, including Android phones and tablets.

Here’s a step-by-step guide to help you create a simple game with **Easel**, and then run it or publish it for use on Android devices:

---

### ✅ Step 1: Understand What Easel Does
Easel is a **browser-based game engine**. It:
- Provides a built-in online editor.
- Compiles your code into HTML5/JavaScript.
- Hosts your game so it's playable instantly in a web browser.
- Automatically handles multiplayer logic and networking.

So while you won’t compile a native `.apk` file like traditional Android apps, you’ll end up with a game that runs in any modern mobile browser — including Android.

---

### ✅ Step 2: Sign Up and Start Coding
1. Go to [https://easel.games](https://easel.games).
2. Click “Start Making Games” or sign up if required.
3. Use the **online editor** to write your game.

You don’t need to install anything — just open your browser (Chrome, Firefox, etc.) on your computer or even directly on an Android device!

---

### ✅ Step 3: Write a Simple Game
Let’s make a basic game where a player moves a square around the screen.

#### Example Code in Easel:
```easel
// Create a player object
player = {
    shape: rectangle(50, 50)
    color: red
    position: (100, 100)

    // Movement behavior
    behavior: {
        every frame: {
            if key(left): position.x -= 200 * dt
            if key(right): position.x += 200 * dt
            if key(up): position.y += 200 * dt
            if key(down): position.y -= 200 * dt
        }
    }
}
```

This creates a red square that moves based on arrow keys. The `dt` multiplier ensures smooth movement across different devices.

> 💡 Tip: You can test this right in the Easel editor by clicking Run.

---

### ✅ Step 4: Publish Your Game
Once you’re happy with your game:
1. Click the **Publish** button in the Easel editor.
2. Easel will generate a public URL for your game.

Example URL:
```
https://easel.games/@yourname/yourgame
```

Now anyone can play your game by visiting that link in their browser — **including Android users!**

---

### ✅ Step 5: Make It Work Like a Native App on Android
If you want your Easel game to behave more like a native app on Android:

#### Option A: Add to Home Screen (PWA-style)
Modern browsers allow you to add websites as shortcuts to your home screen:
1. Open your Easel game in Chrome on Android.
2. Tap the **three dots > Add to Home screen**.
3. It will appear like an app and launch full-screen.

#### Option B: Wrap It in a WebView (Advanced)
To publish on Google Play Store:
1. Use a tool like **Android Studio** to create a simple WebView app.
2. Load your Easel game’s URL inside the WebView.
3. Package it as an APK and upload to the Play Store.

This requires some Java/Kotlin knowledge, but there are many tutorials available online.

---

### ✅ Step 6: Expand Your Game
Want to make something more advanced?
- Add **multiplayer support** automatically — just define how players interact, and Easel handles syncing them.
- Add **UI elements** like score counters or buttons.
- Add **sound effects**, physics, and animations.

E.g., here’s how to add a second player:
```easel
player2 = {
    shape: rectangle(50, 50)
    color: blue
    position: (300, 300)

    behavior: {
        every frame: {
            if key(a): position.x -= 200 * dt
            if key(d): position.x += 200 * dt
            if key(w): position.y += 200 * dt
            if key(s): position.y -= 200 * dt
        }
    }
}
```

Multiple people can join the same game session and control their own characters — all handled automatically by Easel.

---

### 🧠 Summary
| Goal | How to Do It |
|------|--------------|
| Create a game | Use the Easel editor |
| Test it | Click "Run" in the editor |
| Share it | Use the "Publish" button |
| Play on Android | Open the URL in Chrome or add to home screen |
| Turn into an app | Use WebView wrapper (optional) |

---

### 🚀 Final Thoughts
Easel is perfect for making fun, interactive 2D games that work across platforms — including Android. While it doesn’t compile directly to Android APKs, its browser-based output works great on Android via Chrome or as a PWA (Progressive Web App).

If you're new to coding, Easel is one of the best ways to start making real, playable games — especially multiplayer ones — without getting bogged down in complex setup or networking code.

---

Next: write a specific type of game (like a platformer, shooter, or puzzle game)
