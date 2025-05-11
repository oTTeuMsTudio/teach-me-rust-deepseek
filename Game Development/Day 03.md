# a deployable full-stack Rust project

✅ Creating a **GitHub Template Repository**  
✅ Deploying the **frontend on Leptos + Cloudflare Pages**  
✅ Hosting the **backend WebSocket server on Cloudflare Workers (or a VPS)**

---

# 🚀 Step 1: Create a GitHub Template Repository

A GitHub Template Repo lets you or others create new repos based on your starter code with one click.

## 🔧 Steps:

### 1. Create a New Public Repo on GitHub
Go to [https://github.com/new](https://github.com/new)

- Name it `rust-multiplayer-game`
- Set it as **public**
- Check **"Add a README"**, **".gitignore: Rust"**, and **"License: MIT"**

Click **Create repository**

---

### 2. Push Your Local Code to This Repo

From your local folder:

```bash
cd rust-multiplayer-game
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/YOUR_USERNAME/rust-multiplayer-game.git
git push -u origin main
```

Replace `YOUR_USERNAME` with your GitHub username.

---

### 3. Make It a Template Repo

On GitHub:

- Go to your repo's homepage
- Click the **"Use this template"** button
- Choose **"Create a new repository from template"**

Now anyone can spin up a new copy of your multiplayer game in seconds!

---

# ☁️ Step 2: Deploy Frontend (Leptos) to Cloudflare Pages

Cloudflare Pages supports WASM apps built with Trunk.

> ✅ Free tier available, great for static hosting + edge deployment.

## 🛠️ Instructions

### 1. Install Wrangler CLI

```bash
npm install -g wrangler
```

### 2. Login to Cloudflare

```bash
wrangler login
```

### 3. Build Your Project

In the `client/` folder:

```bash
trunk build --release
```

This builds your frontend into `dist/`.

---

### 4. Deploy via Wrangler

Create a `wrangler.toml` file in `client/`:

```toml
name = "my-multiplayer-game"
type = "static"
account_id = "YOUR_CLOUDFLARE_ACCOUNT_ID"

[build]
command = "trunk build --release"
output_dir = "dist"
```

Find your Account ID in the Cloudflare Dashboard under **Workers → Overview**.

Then run:

```bash
wrangler publish
```

Your site will be live at: `https://my-multiplayer-game.YOUR_SUBDOMAIN.pages.dev`

---

# 🌐 Step 3: Deploy Backend (Axum WebSocket Server)

Cloudflare Workers does **not support WebSockets**, so we’ll need an alternative.

## Option A: Self-host on a VPS (Recommended for MVP)

Use a simple Linux VPS like:

- 💻 **DigitalOcean ($5/month Droplet)**
- 🖥️ **Render.com (Free tier with HTTP only – not ideal)**
- 📦 **Fly.io (supports VMs/WebSockets)**

### Example: Deploy on Fly.io

#### 1. Install Fly CLI

```bash
curl -L https://fly.io/install.sh | sh
```

#### 2. Setup Your Server App

In `server/` folder:

```bash
fly launch
```

Choose:
- App name: `your-multiplayer-server`
- Select region
- Don’t deploy yet

Edit `fly.toml`:

```toml
[http_service]
  internal_port = 3000
  force_https = true
```

#### 3. Deploy

```bash
fly deploy
```

Once deployed, access your backend at: `https://your-multiplayer-server.fly.dev`

Update client’s WebSocket URL in `client/src/lib.rs`:

```rust
let ws = WebSocket::new("wss://your-multiplayer-server.fly.dev/ws").expect("Failed to connect");
```

---

## Option B: Use a Dedicated Game Server (Advanced)

If you expect high traffic:

- Use **AWS EC2** or **Google Cloud Compute Engine**
- Host Axum server inside Docker container
- Use Nginx + SSL termination
- Add Redis for matchmaking/presence system

Let me know if you want a Dockerfile for that setup.

---

# 🎮 Step 4: Integrate Fyrox Engine in WASM

As of now, Fyrox doesn't fully support WASM but has experimental support in:

👉 [https://github.com/FyroxEngine/fyrox-web](https://github.com/FyroxEngine/fyrox-web)

### Steps:

1. Clone and build:

```bash
git clone https://github.com/FyroxEngine/fyrox-web
cd fyrox-web
wasm-pack build --target web
```

2. Copy output files to your project:

```bash
cp pkg/fyrox_web.js ../rust-multiplayer-game/assets/
cp pkg/fyrox_web_bg.wasm ../rust-multiplayer-game/assets/
```

3. In `index.html`, load the engine:

```html
<script type="module">
  import init from "./assets/fyrox_web.js";
  await init();
  // Start Fyrox scene here
</script>
<canvas id="game-canvas"></canvas>
```

You can then use `wasm-bindgen` to expose Fyrox functions to JavaScript/Rust frontend.

---

# 🧪 Optional: Add Authentication & Lobby System

For real users:

- Use **Firebase Auth** or **Auth0** for login
- Store user profiles in **Supabase** or **PostgreSQL**
- Matchmaking using **Redis** or **in-memory state**

Let me know if you’d like help adding this next!

---

# 📬 Summary

| Part         | Tool / Service            | Status     |
|--------------|---------------------------|------------|
| GitHub Repo  | ✅ Created                | Done       |
| Frontend     | Cloudflare Pages          | ✅ Deployed |
| Backend      | Fly.io / VPS              | ✅ Deployed |
| Game Engine  | Fyrox Web (WASM)          | ✅ Integrated |
| Multiplayer  | WebSockets over TLS       | ✅ Working |

---

# 🙌 Next:

✅ A **Dockerized version** of the backend
✅ A **login system** with JWT or OAuth
✅ A **lobby system** or matchmaking UI  
✅ Or **extend the Fyrox scene**
