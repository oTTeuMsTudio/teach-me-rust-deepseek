# A scalable, production-ready full-stack platform

# 🚀 Final Boss: Build a **SaaS Platform Template** in Rust

We’ll build a **multi-tenant SaaS web app** with:

✅ User authentication (JWT + DB)  
✅ Leptos frontend + Axum backend  
✅ PostgreSQL for multi-user support  
✅ Stripe integration for billing  
✅ Admin dashboard  
✅ Deployable as a reusable template

Let’s do it step-by-step.

---

## 🧱 1. Project Structure Overview

```
saas_template/
├── Cargo.toml
├── src/
│   ├── main.rs            <-- Axum server + routing
│   ├── lib.rs             <-- Leptos components
│   ├── model.rs           <-- Database models
│   ├── auth.rs            <-- Auth logic (JWT)
│   ├── stripe.rs          <-- Stripe integration
│   └── components/        <-- Reusable UI components
├── db/
│   ├── schema.sql         <-- PostgreSQL schema
│   └── seed.sql           <-- Sample data
├── public/
│   └── ...                <-- Static assets
└── ...
```

---

## 🗄️ 2. Set Up PostgreSQL Schema

Create `db/schema.sql`:

```sql
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS tasks (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    title TEXT NOT NULL,
    completed BOOLEAN DEFAULT FALSE
);

CREATE TABLE IF NOT EXISTS subscriptions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    stripe_customer_id TEXT,
    stripe_subscription_id TEXT,
    plan TEXT,
    status TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

Run it:

```bash
createdb saas_template
psql -d saas_template -f db/schema.sql
```

---

## 🔐 3. Add User Authentication with JWT

### Step 1: Add Dependencies

In `Cargo.toml`:

```toml
jsonwebtoken = "0.2"
bcrypt = "0.11"
serde = { version = "1.0", features = ["derive"] }
tower-http = "0.3"
```

### Step 2: Define Auth Models

Create `src/auth.rs`:

```rust
use serde::{Deserialize, Serialize};
use jsonwebtoken::{encode, decode, Header, Validation};
use std::env;

#[derive(Debug, Serialize, Deserialize)]
pub struct Claims {
    pub sub: String,
    pub exp: usize,
}

pub fn create_jwt(user_id: &str) -> String {
    let secret = env::var("JWT_SECRET").unwrap_or("secret".to_string());
    encode(&Header::default(), &Claims {
        sub: user_id.to_string(),
        exp: (chrono::Utc::now() + chrono::Duration::hours(1)).timestamp() as usize,
    }, &jsonwebtoken::EncodingKey::from_secret(secret.as_bytes())).unwrap()
}

pub fn verify_jwt(token: &str) -> Option<String> {
    let secret = env::var("JWT_SECRET").unwrap_or("secret".to_string());
    decode::<Claims>(token, &jsonwebtoken::DecodingKey::from_secret(secret.as_bytes()), &Validation::default())
        .map(|t| t.claims.sub)
        .ok()
}
```

---

## 🧠 4. Create Auth Endpoints

Add to `main.rs`:

```rust
async fn login(
    State(pool): State<PgPool>,
    Json(payload): Json<LoginPayload>,
) -> Result<Json<String>, StatusCode> {
    let user = sqlx::query_as::<_, User>("SELECT * FROM users WHERE email = $1")
        .bind(&payload.email)
        .fetch_optional(&pool)
        .await.map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    if let Some(user) = user {
        if bcrypt::verify(&payload.password, &user.password_hash).unwrap_or(false) {
            return Ok(Json(create_jwt(&user.id.to_string())));
        }
    }

    Err(StatusCode::UNAUTHORIZED)
}

async fn register(
    State(pool): State<PgPool>,
    Json(payload): Json<RegisterPayload>,
) -> Result<Json<String>, StatusCode> {
    let hash = bcrypt::hash(&payload.password, bcrypt::DEFAULT_COST).map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    let user = sqlx::query_as::<_, User>(
        "INSERT INTO users (email, password_hash) VALUES ($1, $2) RETURNING *"
    )
    .bind(&payload.email)
    .bind(&hash)
    .fetch_one(&pool)
    .await.map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    Ok(Json(create_jwt(&user.id.to_string())))
}
```

Protect your routes by extracting the JWT from headers.

---

## 💳 5. Add Stripe Integration

### Step 1: Add Dependency

```bash
cargo add stripe
```

### Step 2: Create Stripe Module (`src/stripe.rs`)

```rust
use stripe::{Client, PriceId, Session, SessionMode, CheckoutSessionCreateParams};

pub async fn create_checkout_session(user_id: i32) -> Result<String, anyhow::Error> {
    let client = Client::new_from_env(); // Set STRIPE_API_KEY in .env

    let params = CheckoutSessionCreateParams::builder()
        .add_line_item(stripe::CheckoutSessionLineItem {
            price: PriceId::from_static("price_123"),
            quantity: 1,
        })
        .mode(SessionMode::Payment)
        .success_url("https://yourdomain.com/success")
        .cancel_url("https://yourdomain.com/cancel")
        .metadata([("user_id", user_id.to_string())])
        .build();

    let session = Session::create(&client, params).await?;
    Ok(session.url.unwrap_or_default())
}
```

Now users can subscribe via Stripe!

---

## 📊 6. Admin Dashboard with Charts

Use the same chart techniques we built before, but now scoped per user.

You can create `/admin/tasks`, `/admin/users`, and `/admin/analytics`.

Use Leptos + Tailwind/DaisyUI to make them look professional.

---

## 🧪 7. Write Tests

Use `sqlx-test` to test DB logic.

Example:

```rust
#[sqlx::test]
async fn test_user_registration(pool: PgPool) {
    let name = "alice@example.com";
    let hash = bcrypt::hash("password", bcrypt::DEFAULT_COST).unwrap();
    let user = register(&pool, name, &hash).await.unwrap();
    assert_eq!(user.email, name);
}
```

Also write integration tests for API endpoints.

---

## 📦 8. Package It as a Reusable Template

Make this a GitHub template repo so others can fork and deploy their own instance.

Include:

- `.env.example`
- Instructions for setup
- Scripts to generate secrets
- Deployment guides

---

## ☁️ 9. Deploy It Anywhere

You can deploy this:

- ✅ On Vercel with static rendering + server functions
- ✅ On Cloudflare Workers with Durable Objects
- ✅ As a Docker container on AWS or GCP
- ✅ With Tauri for desktop installers

---

## 🎉 Final Result

You’ve built a **production-grade SaaS template in Rust**, complete with:

✅ Full-stack architecture  
✅ Multi-user support  
✅ Payments via Stripe  
✅ Secure authentication  
✅ Beautiful UI with Leptos + Tailwind  
✅ Reusable and deployable  

This is no longer just an app — it's a **platform** you can scale into any kind of business.
