# Create a GitHub repository for Leptos full-stack To-Do app.

---

## 🔐 Step 1: Authenticate with GitHub

To create a repo programmatically use the **GitHub CLI (`gh`)**

### 📦 Install GitHub CLI (if not already)

#### macOS:
```bash
brew install gh
```

#### Linux:
```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-key C99B113F0E74F028D6FEA9DF08FB1FA47DD521CC
sudo apt-add-repository https://cli.github.com/packages
sudo apt update
sudo apt install gh
```

#### Windows (via PowerShell):
Use [Windows installer from GitHub CLI](https://github.com/cli/cli/releases/latest)

---

## 🚀 Step 2: Login to GitHub

Run this in your terminal:

```bash
gh auth login
```

Follow the prompts to authenticate via browser or token.

---

## 🧱 Step 3: Create a New GitHub Repo

Name the repo `leptos-todo-app`.

```bash
gh repo create leptos-todo-app --public -y
```

This creates a public repo named `leptos-todo-app` under your GitHub username.

---

## 📁 Step 4: Initialize Local Git & Push Code

Initialize a local git repo and push the code to GitHub.

```bash
# Initialize git
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit: Leptos full-stack To-Do app"

# Link to remote repo
git branch -M main
git remote add origin https://github.com/<your-username>/leptos-todo-app.git

# Push
git push -u origin main
```

> Replace `<your-username>` with your actual GitHub username.

---

## ✅ GitHub repo working at:

👉 `https://github.com/<your-username>/leptos-todo-app`

It includes:
- Full-stack Rust setup with Leptos + Axum
- SQLite database
- Client-server communication
- Ready-to-extend architecture

