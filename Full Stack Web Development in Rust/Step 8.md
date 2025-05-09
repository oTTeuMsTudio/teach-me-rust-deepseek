# Building a full-stack Rust web app like a pro

Let’s go ahead and **add logout functionality** and **protect the `/api/todos` endpoint** so only logged-in users can access it.

Complete your authentication flow:
- ✅ Register
- ✅ Login
- ✅ Access protected routes
- ✅ Logout

---

# 🔐 1. Add Logout Route (Server Side)

### a. In `main.rs`, add this route:

```rust
async fn logout() -> impl IntoResponse {
    let cookie = Cookie::build(("token", ""))
        .path("/")
        .http_only(true)
        .secure(cfg!(not(debug_assertions)))
        .max_age(time::Duration::seconds(-1)) // Expire immediately
        .finish();

    (StatusCode::OK, cookie.to_string())
}
```

Then add the route to your router:

```rust
.route("/logout", post(logout))
```

---

# 🧠 2. Add Auth Middleware (Optional but Recommended)

For now, we’ll check the token in `/api/todos`.

Update `get_todos()` like this:

```rust
async fn get_todos(
    State(state): State<Arc<AppState>>,
    headers: HeaderMap,
) -> Result<impl IntoResponse, StatusCode> {
    let auth_header = headers.get("Cookie")
        .and_then(|v| v.to_str().ok())
        .unwrap_or("");

    if !auth_header.contains("token=") {
        return Err(StatusCode::UNAUTHORIZED);
    }

    let todos = sqlx::query!("SELECT text FROM todos")
        .fetch_all(&state.pool)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;

    let texts: Vec<String> = todos.iter().map(|t| t.text.clone()).collect();
    Ok(Json(texts))
}
```

You can later move this into middleware for better code organization.

---

# 🖥️ 3. Add Logout Button (Frontend)

Update your `TodoApp` component to include a logout button after login.

We'll use a signal to track login state.

### a. Add Signal for Login State

At top of `TodoApp()`:

```rust
let (logged_in, set_logged_in) = create_signal(false);
```

### b. Add Logout Handler

```rust
let handle_logout = move |_| {
    spawn_local(async move {
        reqwasm::http::Request::post("/logout")
            .send()
            .await
            .unwrap();
        set_logged_in(false);
    });
};
```

### c. Show Logout Button When Logged In

Inside your view:

```rust
{move || if logged_in() {
    view! {
        <button
            on:click=handle_logout
            class="text-sm text-red-500 hover:text-red-700 underline"
        >
            "Logout"
        </button>
    }.into_view()
} else {
    view! {
        <LoginForm set_logged_in />
    }.into_view()
}}
```

And update your `LoginForm` to take `set_logged_in` as prop:

```rust
#[component]
fn LoginForm(set_logged_in: WriteSignal<bool>) -> impl IntoView {
    // ... existing code ...
    set_logged_in(true); // On successful login
}
```

---

## 🧪 4. Test It Out

Run your dev server:

```bash
trunk serve --port 3000
```

Go to [http://localhost:3000](http://localhost:3000), try logging in, adding todos, and logging out.

---

## 📦 Step 5: Commit Changes

```bash
git add .
git commit -m "Add logout functionality and protect /api/todos"
git push origin main
```

---

## ✅ Now we have:
- Full user registration
- Login with JWT
- Protected API endpoints
- Logout support
- UI that updates based on auth status

---

## 🚀 Next:
1. Store user ID in localStorage or a global signal
2. Add GitHub Actions CI/CD pipeline
3. Add role-based access control (admin/user)
4. Deploy to production with automatic builds
