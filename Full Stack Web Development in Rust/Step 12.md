# Building a full-stack Rust web app with:
- Leptos frontend
- Axum backend
- SQLite + D1 database
- JWT authentication
- GitHub Actions CI/CD
- End-to-end browser tests

Now let’s **add custom domain and SSL support** so your app can go live on a professional URL like:

👉 `https://app.yourdomain.com`

---

## 🌐 Step 1: Buy a Custom Domain

If you haven’t already, buy a domain from one of these providers:
- 🔹 [Namecheap](https://www.namecheap.com/)
- 🔹 [Cloudflare Registrar](https://www.cloudflare.com/registrar/)
- 🔹 [Google Domains](https://domains.google/)
- 🔹 [GoDaddy](https://www.godaddy.com/)

We'll use **Cloudflare** for DNS and free SSL.

---

## ☁️ Step 2: Add Your Domain to Cloudflare

1. Go to [https://dash.cloudflare.com](https://dash.cloudflare.com)
2. Click **Add a site**
3. Enter your domain (e.g., `yourdomain.com`)
4. Select the **Free Plan**
5. Follow instructions to update nameservers at your registrar

Once done, Cloudflare becomes your DNS provider.

---

## 🧱 Step 3: Configure DNS Records in Cloudflare

Go to **DNS** tab in Cloudflare and add:

### A Record for Root Domain (optional)
| Type | Name     | Content         | Proxy status |
|------|----------|------------------|--------------|
| A    | @        | YOUR_SERVER_IP   | Proxied      |

> If you're hosting on Workers or Pages, skip this step.

### CNAME for Subdomain (e.g., `app.yourdomain.com`)
| Type  | Name       | Target                     | Proxy status |
|-------|------------|----------------------------|--------------|
| CNAME | app        | `.pages.dev` or `.workers.dev` | Proxied      |

Example:
```
app.yourdomain.com → leptos-todo.pages.dev
```

---

## 🔒 Step 4: Enable SSL/TLS

Go to **SSL/TLS** tab → **Overview**

Set to:
✅ **Full (strict)**

This enables HTTPS for all traffic.

Also enable:
- Always Use HTTPS
- HTTP to HTTPS redirect

---

## 🚀 Step 5: Update Worker / Pages Deployment (Optional)

If you’re using Cloudflare Pages or Workers:

### a. In `wrangler.toml` (Workers)

```toml
name = "leptos-todo-api"
route = "app.yourdomain.com/*"
zone_id = "your-zone-id"
```

Or if using Pages:

Go to **Pages project settings** → **Custom domains**  
Add `app.yourdomain.com` and wait for verification

---

## 🖥️ Step 6: Update Frontend Base URLs

In your Leptos app (`lib.rs` or wherever you make API calls), update any hardcoded URLs to point to your domain:

```rust
let api_base = "https://api.yourdomain.com";
```

Then rebuild and redeploy.

---

## 🧪 Step 7: Test It Out

Visit:
👉 `https://app.yourdomain.com`

You should see:
✅ Your To-Do app  
✅ Green padlock ✅  
✅ No certificate errors  

---

# 📦 Done! Your App Is Now Live on a Custom Domain with SSL

✅ Custom domain configured  
✅ Free SSL via Cloudflare  
✅ Secure HTTPS enforced  
✅ All set for production

---

## 🎯 Next:
1. Set up **logging & error tracking (Sentry or logflare)**
2. Add **analytics dashboard (Plausible or PostHog)**
3. Add **email signup/password reset flow**
4. Create a **Dockerfile for local dev/prod builds**

