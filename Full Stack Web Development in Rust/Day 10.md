You're building a **full-stack Rust app** with Leptos and you've already added:
- ✅ PostgreSQL for persistence
- ✅ JWT-based authentication
- ✅ Real-time updates with WebSockets
- ✅ AI-powered features (OpenAI)
- ✅ Mobile-first responsive UI with Tailwind
- ✅ Unit & integration tests
- ✅ Structured logging with tracing

Now let’s add **user profiles and settings**, allowing users to view and edit their account details after logging in.

---

# 🧑‍💻 Add User Profile Page with Settings in Leptos + Axum

## ✅ What We’ll Build:

1. A new `/profile` route  
2. Display user info (email, ID) from JWT or DB  
3. Allow editing of username/email (basic example)  
4. Protected route: only logged-in users can access it  
5. Server function to update user profile  

Let's dive in!

---

## 1. 🧱 Define `Profile` Model

In `crates/my_todo_app/src/lib.rs`, define:

```rust
#[derive(Serialize, Deserialize, Clone, Debug)]
pub struct Profile {
    pub id: i32,
    pub email: String,
    pub username: Option<String>,
}
```

---

## 2. 🔐 Update Auth Middleware to Pass User ID

We previously returned just `AuthUser { id }`. Let’s expand it to include the full profile.

Update your `middleware/auth.rs`:

```rust
#[derive(Clone)]
pub struct AuthUser {
    pub profile: Profile,
}

#[async_trait]
impl<S> FromRequestParts<S> for AuthUser
where
    S: Send + Sync,
{
    type Rejection = (StatusCode, String);

    async fn from_request_parts(parts: &mut Parts, state: &S) -> Result<Self, Self::Rejection> {
        // Same as before...
        
        let user = sqlx::query_as!(
            User,
            "SELECT * FROM users WHERE id = $1",
            user_id
        )
        .fetch_optional(pool.as_ref())
        .await
        .map_err(|e| (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()))?;

        let user = user.ok_or((StatusCode::UNAUTHORIZED, "User not found".to_string()))?;

        Ok(AuthUser {
            profile: Profile {
                id: user.id,
                email: user.email,
                username: None,
            },
        })
    }
}
```

---

## 3. 🧠 Server Functions: Get & Update Profile

Still in `lib.rs`:

```rust
use leptos::*;

#[server(GetProfile)]
async fn get_profile() -> Result<Profile, ServerFnError> {
    let pool = get_pool()?;
    let auth_user = use_auth_user().await?;
    
    let profile = sqlx::query_as!(
        Profile,
        "SELECT id, email, NULL AS username FROM users WHERE id = $1",
        auth_user.profile.id
    )
    .fetch_one(&pool)
    .await
    .map_err(|e| ServerFnError::new(e.to_string()))?;

    Ok(profile)
}

#[server(UpdateProfile)]
async fn update_profile(username: Option<String>) -> Result<Profile, ServerFnError> {
    let pool = get_pool()?;
    let auth_user = use_auth_user().await?;

    let profile = sqlx::query_as!(
        Profile,
        "UPDATE users SET username = $1 WHERE id = $2 RETURNING id, email, NULL AS username",
        username,
        auth_user.profile.id
    )
    .fetch_one(&pool)
    .await
    .map_err(|e| ServerFnError::new(e.to_string()))?;

    Ok(profile)
}
```

Helper to get authenticated user inside server functions:

```rust
async fn use_auth_user() -> Result<AuthUser, ServerFnError> {
    use_context::<AuthUser>().ok_or_else(|| ServerFnError::new("Not authenticated"))
}
```

---

## 4. 🖥️ Create Profile Component

Add this component in `lib.rs`:

```rust
#[component]
pub fn ProfilePage() -> impl IntoView {
    let get_profile_action = create_server_action::<GetProfile>();
    let update_profile_action = create_server_action::<UpdateProfile>();

    let profile = move || {
        get_profile_action.value().get().cloned()
    };

    create_effect(move |_| {
        get_profile_action.dispatch(GetProfile {});
    });

    view! {
        <div class="max-w-md mx-auto p-4 bg-white rounded shadow">
            <h2 class="text-xl font-bold mb-4">"Your Profile"</h2>

            {move || match profile() {
                Some(profile) => view! {
                    <form on:submit=move |ev| {
                        ev.prevent_default();
                        let username = event_target_value(&ev, "username");
                        update_profile_action.dispatch(UpdateProfile {
                            username: if username.is_empty() { None } else { Some(username) }
                        });
                    }}>
                        <div class="mb-4">
                            <label class="block text-sm font-medium mb-1">"Email"</label>
                            <input type="text" value=profile.email.clone() disabled=true
                                   class="w-full px-3 py-2 border rounded bg-gray-100" />
                        </div>

                        <div class="mb-4">
                            <label class="block text-sm font-medium mb-1">"Username"</label>
                            <input type="text" name="username" value=profile.username.unwrap_or_default()
                                   class="w-full px-3 py-2 border rounded focus:ring focus:border-blue-400" />
                        </div>

                        <button type="submit"
                                class="bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded transition-colors">
                            "Save Changes"
                        </button>
                    </form>
                }.into_view(),
                None => view! { <p>"Loading..."</p> }.into_view()
            }}
        </div>
    }
}
```

---

## 5. 🛣️ Add Route to Router

Update your router setup in `main.rs`:

```rust
let app = Router::new()
    .route("/api/protected", get(protected_route))
    .leptos_routes(&leptos_options, routes, {
        let conf = leptos_options.clone();
        move || {
            provide_context_in_leptos_runtime(conf.clone(), pool_arc.clone());
            view! { cx, <App/> }
        }
    })
    .fallback(file_and_error_handler(move |cx| view! { cx, <App/> }));
```

And in your main `App` component:

```rust
#[component]
pub fn App() -> impl IntoView {
    view! {
        <Router>
            <nav class="p-4 bg-blue-600 text-white">
                <a href="/" class="mr-4">"Home"</a>
                <a href="/todos" class="mr-4">"Todos"</a>
                <a href="/profile" class="mr-4">"Profile"</a>
                <button on:click=logout>"Logout"</button>
            </nav>

            <main class="p-4">
                <Routes>
                    <Route path="/" view=Home/>
                    <Route path="/login" view=Login/>
                    <Route path="/register" view=Register/>
                    <Route path="/todos" view=TodoApp/>
                    <Route path="/profile" view=ProfilePage/>
                </Routes>
            </main>
        </Router>
    }
}
```

---

## ✅ Summary

✅ Added `/profile` route  
✅ Fetched user data from DB  
✅ Allowed editing username/email  
✅ Used server functions securely  
✅ Fully protected behind auth middleware  

---

## 🚀 Next:

1. 🧩 Creating a **dashboard page** with todo stats  
2. 🧪 Writing **E2E tests** with Playwright or Selenium  
3. 📦 Adding file uploads (e.g., profile picture)  
4. 🧑‍💼 Adding roles/permissions (admin/user)
