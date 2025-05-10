# Building a full-stack Rust web app with real-world features

# Deploy your Leptos + Axum + Postgres + Auth app to a live environment

---

# ☁️ Deploy Now Full-Stack Rust App (with Auth & DB)

We’ll use:

| Service | Purpose |
|--------|---------|
| **Render.com** | Host the Axum backend with Leptos frontend |
| **PostgreSQL** | Use Render’s free PostgreSQL instance |
| **GitHub** | Connect for automatic deploys |

> Why Render? It’s free, supports Rust binaries, and gives you a PostgreSQL DB in one click.

---

## 1. 🧳 Prerequisites

✅ GitHub repo of your project  
✅ Local working version (`trunk serve` works)  
✅ Git installed  
✅ Basic knowledge of deploying apps  

---

## 2. 📦 Prepare for Deployment

### Set up `Cargo.toml` for deployment

Ensure your root `Cargo.toml` includes all dependencies needed for production.

Also ensure you have a `[workspace]` section like this:

```toml
[workspace]
members = [
    "crates/my_todo_app",
    "crates/my_todo_app_axum"
]
```

### Add `.cargo/config.toml` to help build properly

```toml
[build]
target = "x86_64-unknown-linux-gnu"
```

This ensures compatibility with Linux-based deploy environments.

---

## 3. 🌐 Configure Environment Variables

In `my_todo_app_axum/src/main.rs`, we used:

```rust
std::env::var("DATABASE_URL").expect("DATABASE_URL must be set")
```

So we'll need to provide that at runtime.

Create a `.env` file locally if not already there:

```env
DATABASE_URL=postgres://postgres:password@localhost:5432/todo?sslmode=disable
JWT_SECRET=my-super-secret-key
```

But in production, you’ll replace these with secrets on Render.

---

## 4. 🗂️ Update `index.html`

Make sure your `index.html` points to the correct output path:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Leptos Todo App</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
</head>
<body>
  <div id="main"></div>
  <script type="module" src="/main.js"></script>
</body>
</html>
```

---

## 5. 🧪 Test Build Locally

Before deploying, test the release build:

```bash
trunk build --release
```

If it builds without error, we’re good to go!

---

## 6. 🚀 Step-by-Step: Deploy to Render

### ✅ Step 1: Push Code to GitHub

Push the project to a public or private GitHub repo.

Example:
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:yourusername/your-leptos-app.git
git push -u origin main
```

---

### ✅ Step 2: Create PostgreSQL DB on Render

Go to [https://dashboard.render.com/select-new](https://dashboard.render.com/select-new)

Click “PostgreSQL” → “Create New PostgreSQL Database”

- Name: `todo-db`
- Region: Pick closest
- Plan: Free
- Click “Create Database”

After creation, copy:
- **Internal Database URL**
- Or note down:
  - Host
  - Port
  - User
  - Password
  - Database name

---

### ✅ Step 3: Create Web Service on Render

Go back to dashboard → New Web Service

#### Fill in:
- **Name**: `leptos-todo-app`
- **Repo**: Select your GitHub repo
- **Branch**: `main`
- **Root Directory**: `/` (or where your `Cargo.toml` is)
- **Runtime**: Rust
- **Build Command**:
```bash
cargo build --release
```
- **Start Command**:
```bash
./target/release/my_todo_app_axum
```

> Replace `my_todo_app_axum` with the actual binary name from your `Cargo.toml` (under `[[bin]]` or `name` field)

---

### ✅ Step 4: Set Environment Variables

In Render under your service → Secrets / Environment Variables:

Add these:

| Key             | Value                      |
|------------------|----------------------------|
| `DATABASE_URL`   | From Render DB page        |
| `JWT_SECRET`     | A strong secret key (e.g., `super-secret-key-123`) |
| `LEPTOS_OUTPUT_PATH` | `/opt/render/project/src/dist` |
| `TRUNK_TMPDIR`   | `/tmp/trunk`               |

⚠️ Make sure the `DATABASE_URL` matches the format:

```
postgres://user:password@host:port/dbname?sslmode=require
```

---

### ✅ Step 5: Wait for Build

It will take ~3–5 minutes to build the Rust binary and deploy.

Once deployed, open the live site.

---

## 🔍 Optional: Check Logs

Under your service → Logs tab

If it fails:
- Check for missing dependencies
- Ensure all features are enabled in `Cargo.toml`
- Confirm database connection string is correct

---

## 🎉 Full-Stack Rust App is now deployed!

We now have a live Rust app built with:

- ✅ Leptos (frontend + server functions)
- ✅ Axum (backend)
- ✅ Postgres (database)
- ✅ JWT auth
- ✅ Protected routes
- ✅ GitHub + Render deployment

---

## 🚀 Next:

1. 📱 Add mobile responsiveness
2. 📊 Add analytics or logging
3. 🧠 Add AI integration (e.g., GPT-powered todo suggestions)
4. 💬 Add WebSocket chat or real-time updates
5. 🔁 Automate CI/CD with GitHub Actions
