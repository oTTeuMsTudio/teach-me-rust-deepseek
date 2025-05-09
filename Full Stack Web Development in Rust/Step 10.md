# Building a world-class full-stack Rust app with CI/CD, auth, and modern tooling

Let’s **add automated deployment to Cloudflare Workers** using GitHub Actions.

This will:
- ✅ Build your frontend (Leptos + WASM)
- ✅ Deploy the backend Worker
- ✅ Keep everything in sync on every push to `main`

---

## 🧱 Overview

We’ll create a second GitHub Action workflow that:
1. Builds the frontend (`trunk`)
2. Deploys the Worker (`wrangler`)
3. Optionally runs migrations or seeds data

---

# 🛠️ Step 1: Install Wrangler CLI in CI

Cloudflare’s `wrangler` CLI is needed to deploy Workers.

Update your existing `.github/workflows/ci.yml` or create a new one:

### Create `.github/workflows/deploy.yml`

```yaml
name: Deploy to Cloudflare

on:
  push:
    branches: ["main"]

jobs:
  deploy:

    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'

    - name: Install Wrangler
      run: npm install -g wrangler

    - name: Authenticate with Cloudflare
      run: echo "CLOUDFLARE_API_TOKEN=${{ secrets.CLOUDFLARE_API_TOKEN }}" > ~/.wrangler/config.toml
      env:
        CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}

    - name: Setup Rust
      uses: rust-lang/setup-rust@v1
      with:
        toolchain: stable
        override: true

    - name: Install wasm-pack
      run: cargo install wasm-pack

    - name: Install Trunk
      run: cargo install trunk

    - name: Build Frontend
      run: trunk build --release

    - name: Deploy Worker
      working-directory: ./leptos-todo-api
      run: |
        wrangler publish
```

> Replace `./leptos-todo-api` with the correct path if you have a nested structure.

---

# 🔐 Step 2: Add Cloudflare API Token as Secret

Go to:
👉 [https://dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens)

Create a token with:
- ✅ Account: Workers KV & D1 access
- ✅ Zone: DNS (optional)

Then add it to your repo's secrets:
GitHub → Settings → Secrets and variables → Actions → New repository secret

Name: `CLOUDFLARE_API_TOKEN`  
Value: Your generated token

---

# 📁 Step 3: Prepare Your Worker for CI

Make sure your Worker project has:

- `wrangler.toml`
- Valid bindings to D1 DB
- A `src/index.ts` file for the Worker logic

If you followed earlier steps, this should already be done 👍

---

# 🚀 Done! Now Every Push to `main` Will Auto-Deploy

✅ Frontend builds via Trunk  
✅ Worker publishes via Wrangler  
✅ All from GitHub Actions  
✅ No manual deploys required

---

## 🎯 Next:
1. Add **database migrations in CI**
2. Set up **custom domains**
3. Add **end-to-end browser tests**
4. Add **analytics or logging**
