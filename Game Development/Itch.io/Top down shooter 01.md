Awesome! Let’s build a **top-down shooter game using Easel**, and then I’ll show you how to export it and publish it on **itch.io**.

We’ll make a simple version with:
- A player that moves with arrow keys or WASD
- Shooting bullets with the mouse
- Enemies that spawn and move toward the player
- Basic collision detection

---

## 🧱 Step 1: Set Up Your Game in Easel

Go to [https://easel.games](https://easel.games) and sign up or log in. Then:

1. Click **“New Game”**
2. Choose **HTML5 Canvas Template**
3. Name your game something like **TopDownShooter**

---

## 🕹️ Step 2: Create Player Movement

### Add a Player Sprite
- Go to **Assets > Sprites > Add New Sprite**
- Draw a small square or circle (or upload an image)
- Name it `player`

### Add Player Script
Go to **Scripts > Add New Script**  
Name it `PlayerController.js` and paste this code:

```javascript
const speed = 200;

function update() {
    let dx = 0;
    if (input.isKeyPressed("KeyA") || input.isKeyPressed("ArrowLeft")) dx -= 1;
    if (input.isKeyPressed("KeyD") || input.isKeyPressed("ArrowRight")) dx += 1;

    let dy = 0;
    if (input.isKeyPressed("KeyW") || input.isKeyPressed("ArrowUp")) dy -= 1;
    if (input.isKeyPressed("KeyS") || input.isKeyPressed("ArrowDown")) dy += 1;

    let angle = Math.atan2(dy, dx);
    if (dx !== 0 || dy !== 0) {
        this.angle = angle;
        this.x += Math.cos(angle) * speed * Time.deltaTime;
        this.y += Math.sin(angle) * speed * Time.deltaTime;
    }
}
```

### Attach Script to Player
- Select your player sprite
- In the properties panel, under **Script**, select `PlayerController.js`

---

## 🔫 Step 3: Add Shooting Mechanic

### Create Bullet Sprite
- Add a new sprite called `bullet`
- Make it a small red dot or rectangle

### Create Bullet Script
Add a new script called `BulletBehavior.js`:

```javascript
const bulletSpeed = 500;

function update() {
    this.x += Math.cos(this.angle) * bulletSpeed * Time.deltaTime;
    this.y += Math.sin(this.angle) * bulletSpeed * Time.deltaTime;

    // Destroy bullet if off-screen
    if (this.x < -10 || this.x > Scene.width + 10 || this.y < -10 || this.y > Scene.height + 10) {
        this.destroy();
    }
}
```

### Create Gun Script for Player
Create another script: `GunController.js`

```javascript
let fireRate = 0.2; // seconds between shots
let lastShotTime = 0;

function update() {
    if (input.isMouseButtonPressed(0)) {
        if (Time.now - lastShotTime > fireRate * 1000) {
            let b = new GameObject("bullet");
            b.setPosition(this.x, this.y);
            b.angle = Camera.getMouseAngle();
            b.addScript("BulletBehavior.js");
            lastShotTime = Time.now;
        }
    }
}
```

Attach both `GunController.js` and `BulletBehavior.js` to the bullet.

---

## 👾 Step 4: Add Simple Enemy AI

### Create Enemy Sprite
- Add a new sprite named `enemy`
- Give it a color (like green)

### Create Enemy AI Script
Create a script called `EnemyAI.js`:

```javascript
const enemySpeed = 100;

function update() {
    let dx = Scene.findGameObjectWithTag("player").x - this.x;
    let dy = Scene.findGameObjectWithTag("player").y - this.y;
    let angle = Math.atan2(dy, dx);

    this.angle = angle;
    this.x += Math.cos(angle) * enemySpeed * Time.deltaTime;
    this.y += Math.sin(angle) * enemySpeed * Time.deltaTime;
}
```

Attach it to the enemy.

---

## 💥 Step 5: Add Collision Detection

### Detect Bullet-Enemy Collisions
Create a new script called `CollisionManager.js`:

```javascript
function update() {
    let bullets = Scene.findGameObjectsWithTag("bullet");
    let enemies = Scene.findGameObjectsWithTag("enemy");

    for (let b of bullets) {
        for (let e of enemies) {
            if (b.collidesWith(e)) {
                e.destroy();
                b.destroy();
            }
        }
    }
}
```

Add this script to any object (you can attach it to a blank GameObject or the scene).

---

## 🐣 Step 6: Spawn Enemies Over Time

Create a script called `EnemySpawner.js`:

```javascript
let spawnInterval = 3; // seconds
let lastSpawnTime = 0;

function update() {
    if (Time.now - lastSpawnTime > spawnInterval * 1000) {
        let e = new GameObject("enemy");
        e.setPosition(Math.random() * Scene.width, Math.random() * Scene.height);
        e.addScript("EnemyAI.js");
        lastSpawnTime = Time.now;
    }
}
```

Attach it to the scene or a dummy object.

---

## ✅ Step 7: Test and Polish

Now test your game:
- Move around with WASD or Arrow Keys
- Shoot with left mouse click
- Enemies should spawn and chase the player
- Bullets destroy enemies on impact

You can now add:
- Health system
- Score counter
- Powerups
- Background music or SFX

---

## 💾 Step 8: Export as HTML5

Once you're happy with your game:

1. Go to **File > Export**
2. Download the ZIP file containing your game
3. Extract it to a folder

---

## 🧨 Step 9: Upload to Itch.io

1. Go to [itch.io](https://itch.io)
2. Sign up or log in
3. Click “Create New Game”
4. Upload your exported ZIP
5. Fill in:
   - Title: Top Down Shooter
   - Description: Describe gameplay, controls, etc.
   - Cover Image: Use a screenshot from your game
   - Tags: #shooter #topdown #action
6. Choose free or set a price
7. Click “Publish”

🎉 Congrats! You’ve made and published a top-down shooter!

---

## 📦 Want More?

Would you like:
- A downloadable ZIP template?
- Help adding sound effects or background music?
- Instructions to add multiplayer support via Easel?

Let me know and I’ll help you go further!
