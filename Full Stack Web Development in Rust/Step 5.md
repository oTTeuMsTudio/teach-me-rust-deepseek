# Deploy full-stack Rust Leptos app to Cloudflare Workers + D1 — a perfect pairing for deploying Rust apps with serverless functions and SQLite-compatible database support.

---

## ☁️ Why Cloudflare Workers + D1?

- ✅ Serverless edge compute (Workers)
- ✅ SQLite-compatible DB (D1)
- ✅ Global CDN
- ✅ No infrastructure to manage
- ✅ Free tier available

---

# 🧱 Step 1: Prepare Your App for Deployment

We’ll:
1. Split frontend and backend
2. Build the frontend as static files
3. Deploy the backend as a Worker with D1

> We'll use `wrangler`, the CLI for Cloudflare Workers.

---

## 🔧 Step 2: Install Wrangler

```bash
npm install -g wrangler
```

Then log in:

```bash
wrangler login
```

Follow the prompts to authorize with your browser.

---

## 🗂️ Step 3: Create Two Projects

### a. Frontend (Leptos Static Site)

Build it using Trunk:

```bash
trunk build --dist dist
```

This generates HTML/JS/WASM in the `dist/` folder.

We’ll deploy this via Workers or host it on Cloudflare Pages.

### b. Backend (Axum API → Cloudflare Worker)

We need to convert our Axum backend into a Worker-compatible format.

> Unfortunately, Axum doesn’t run directly on Workers. But we can write a small Worker that connects to D1 and exposes the same `/api/todos` endpoints.

---

## 🌐 Step 4: Create a New Cloudflare Worker Project

```bash
wrangler init leptos-todo-api
cd leptos-todo-api
```

Replace the contents of `src/index.ts` with:

```ts
export interface Env {
  DB: D1Database;
}

export default {
  async fetch(
    request: Request,
    env: Env,
    ctx: ExecutionContext
  ): Promise<Response> {
    const url = new URL(request.url);

    if (url.pathname === "/api/todos") {
      if (request.method === "GET") {
        const todos = await env.DB.prepare("SELECT text FROM todos")
          .all<{ text: string }>();

        return Response.json(todos.results);
      }

      if (request.method === "POST") {
        const { text } = await request.json<{ text: string }>();
        await env.DB.prepare("INSERT INTO todos (text) VALUES (?)")
          .bind(text)
          .run();

        return new Response("OK", { status: 201 });
      }

      if (request.method === "DELETE") {
        await env.DB.prepare("DELETE FROM todos").run();
        return new Response("Deleted all todos");
      }
    }

    return new Response("Not found", { status: 404 });
  },
};
```

---

## 📦 Step 5: Create a D1 Database

```bash
wrangler d1 create leptos-todo-db
```

Then update your `wrangler.toml`:

```toml
[[d1_databases]]
binding = "DB"
database_name = "leptos-todo-db"
database_id = "your-database-id" # get from output above
```

Then create the table:

```bash
wrangler d1 execute leptos-todo-db --command="CREATE TABLE IF NOT EXISTS todos (id INTEGER PRIMARY KEY AUTOINCREMENT, text TEXT NOT NULL);"
```

---

## 🚀 Step 6: Deploy the Worker

```bash
wrangler publish
```

This will give you a URL like:

👉 `https://leptos-todo-api.<your-subdomain>.workers.dev`

---

## 🖥️ Step 7: Update Frontend to Use Worker API

In your `src/lib.rs` file, change the base URL for requests:

```rust
let api_base = "https://leptos-todo-api.<your-subdomain>.workers.dev";
```

Update both GET and POST URLs accordingly:

```rust
reqwasm::http::Request::get(&format!("{}/api/todos", api_base))
reqwasm::http::Request::post(&format!("{}/api/todos", api_base))
```

Then rebuild:

```bash
trunk build
```

---

## 🌍 Step 8: Deploy Frontend (Optional)

You can deploy the `dist/` folder to:

- ✅ [Cloudflare Pages](https://pages.cloudflare.com/)
- ✅ [Vercel](https://vercel.com/)
- ✅ [Netlify](https://netlify.com)

For Cloudflare Pages:

```bash
wrangler pages project deploy dist --name leptos-todo-app
```

Now you have:

✅ Frontend: `https://leptos-todo-app.pages.dev`  
✅ Backend: `https://leptos-todo-api.<your-subdomain>.workers.dev`  
✅ Database: D1

---

## 🎉 Full-stack Rust web app built with:

- 🦀 Rust + Leptos
- ⚡ Cloudflare Workers
- 💾 D1 Database
- 🌐 Edge deployment

---
