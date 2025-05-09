# Set up a GitHub Actions CI/CD pipeline** for full-stack Rust Leptos app.

This will automate:
- ✅ Building the frontend (`trunk build`)
- ✅ Testing your Rust code
- ✅ Linting (clippy)
- ✅ Deploying to Cloudflare Workers (optional, advanced)

---

# 🧰 What We’ll Do

We'll create a `.github/workflows/ci.yml` file that runs on every push and PR. It will:

1. 🔧 Set up Rust environment
2. 📦 Install dependencies
3. 🧪 Run tests and clippy linting
4. 🛠️ Build the frontend with Trunk
5. 📤 (Optional) Upload artifacts or deploy

---

# 📁 Step 1: Create `.github/workflows/ci.yml`

Add this to your repo:

```yaml
name: Rust CI

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup Rust
      uses: rust-lang/setup-rust@v1
      with:
        toolchain: stable
        override: true

    - name: Install wasm-pack
      run: cargo install wasm-pack

    - name: Install Trunk
      run: cargo install trunk

    - name: Install Clippy
      run: rustup component add clippy

    - name: Install SQLx CLI
      run: cargo install sqlx-cli --features sqlite

    - name: Setup SQLite DB
      run: |
        mkdir -p db
        touch db/todos.db
        sqlite3 db/todos.db < db/schema.sql

    - name: Build Project
      run: |
        trunk build --release

    - name: Run Tests
      run: |
        cargo test --workspace --all-targets

    - name: Run Clippy
      run: |
        cargo clippy --workspace --all-targets -- -D warnings
```

---

# 🧪 Step 2: Add `Cargo.toml` Workspace (if not already set up)

If you have multiple crates/modules, define a workspace in root `Cargo.toml`:

```toml
[workspace]
members = ["src"]
```

Or just keep it simple if you’re using a single crate.

---

# 🧼 Step 3: Commit and Push

```bash
git add .github/workflows/ci.yml
git commit -m "Add GitHub Actions CI pipeline"
git push origin main
```

---

# 🚀 Done!

✅ Now, every time you push to `main` or open a PR:
- Your app builds
- All tests run
- Rust lints are checked

This ensures quality and helps prevent breaking changes from slipping into production.

---

## 🎯 Next:
1. Set up **automated deployment to Cloudflare Workers**
2. Add **database migrations in CI**
3. Set up **linting + formatting (rustfmt)**
4. Add **browser testing (e.g., Playwright)**
