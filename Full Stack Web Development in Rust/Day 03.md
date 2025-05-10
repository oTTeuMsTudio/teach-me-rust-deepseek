# Securely store the JWT, implement redirects after login, and make sure users only see the `/todos` page when logged in

---

# 🔐 Part 2: Secure Auth Flow with Redirects & Protected UI

We'll cover:
✅ Save JWT to `localStorage`  
✅ Redirect user after login/register  
✅ Protect routes with middleware  
✅ Show/hide parts of UI based on auth status  

Let’s dive in!

---

## 1. 🗃️ Store JWT in Browser Storage (Client-side)

We’ll use `web_sys::window().unwrap().local_storage()` to access localStorage.

Add this inside your login success handler:

```rust
use leptos::wasm_bindgen::JsCast;
use web_sys::Storage;

// Inside your App component
let login_action = create_server_action::<Login>();

create_effect(move |_| {
    if let Some(Ok(token)) = login_action.value().get() {
        let window = web_sys::window().unwrap();
        let storage: Storage = window.local_storage().unwrap().unwrap();
        storage.set_item("token", &token).unwrap();
        
        // Redirect to /todos
        set_href("/todos");
    }
});
```

Helper function for redirect:

```rust
#[inline]
fn set_href(href: &str) {
    let window = web_sys::window().unwrap();
    window.location().set_href(href).unwrap();
}
```

Do the same in `register_action` too if you want auto-login after registration.

---

## 2. 🔒 Protect Routes (Only Logged-in Users Can See Todos)

### Step A: Create a new protected route

Update your frontend component to check auth state.

In `lib.rs`, add a helper to read token from localStorage:

```rust
fn get_token() -> Option<String> {
    let window = web_sys::window().unwrap();
    window.local_storage().ok().flatten().and_then(|storage| {
        storage.get_item("token").ok().flatten()
    })
}
```

Then wrap your todo UI in a conditional:

```rust
#[component]
pub fn TodoApp() -> impl IntoView {
    let token = get_token();

    if token.is_none() {
        return view! { <Redirect path="/login"/> };
    }

    // Otherwise show todos
    view! {
        <div>
            <h1>"Your Todos"</h1>
            {/* Your todo list logic here */}
        </div>
    }
}
```

Helper `Redirect` component:

```rust
#[component]
fn Redirect(path: &'static str) -> impl IntoView {
    create_effect(move |_| {
        set_href(path);
    });
    ().into_view()
}
```

---

## 3. 🛣️ Set Up Routes (Leptos Router)

Use Leptos Router to define multiple pages.

Install leptos router:

```bash
cargo add leptos_router
```

Then update `main.rs` or `lib.rs` to define routes:

```rust
use leptos_router::*;

#[component]
pub fn App() -> impl IntoView {
    view! {
        <Router>
            <Routes>
                <Route path="/" view=Home/>
                <Route path="/login" view=Login/>
                <Route path="/todos" view=TodoApp/>
            </Routes>
        </Router>
    }
}
```

Now can build separate components for each page.

---

## 4. 👤 Pass Auth Token in Server Function Headers

By default, server functions don’t include headers like Authorization.

To fix that, override the fetcher:

```rust
provide_context(cx, ServerFnWrapper::new_with_fetcher(move |input| {
    let token = get_token();
    async move {
        let mut req = Request::new(Method::POST, input.url());
        req.headers_mut()
            .insert("Authorization", format!("Bearer {}", token.as_deref().unwrap_or("")).parse().unwrap());

        req.body_mut().push_bytes(input.body().as_bytes()).unwrap();

        let resp = fetch(req).await?;
        Ok(resp.text().await?)
    }
}));
```

This ensures every server function call includes the bearer token.

---

## 5. 🧹 Logout Functionality

Add a logout button somewhere:

```rust
<button on:click=move |_| {
    let window = web_sys::window().unwrap();
    let storage = window.local_storage().unwrap().unwrap();
    storage.remove_item("token").unwrap();
    set_href("/");
}>
    "Logout"
</button>
```

---

## ✅ Summary: We Now Have

- ✅ JWT stored in `localStorage`
- ✅ Login redirects to `/todos`
- ✅ Protected routes using Leptos Router
- ✅ Server functions send auth headers
- ✅ Logout functionality

---

## 🚀 Next:

1. **Deploying** this full-stack app securely
2. Adding **refresh tokens** and session expiration handling
3. Using **OAuth (Google, GitHub)** instead of email/password
4. Building a **dashboard or profile page**
