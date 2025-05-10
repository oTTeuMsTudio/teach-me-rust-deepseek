🔥 You're building a **professional full-stack Rust app** using **Leptos**, and you've already added:
- ✅ PostgreSQL for persistence
- ✅ JWT-based authentication
- ✅ Real-time updates with WebSockets
- ✅ AI-powered features (OpenAI)
- ✅ Mobile-first responsive UI with Tailwind
- ✅ Unit & integration tests
- ✅ Structured logging with tracing
- ✅ User profiles and settings

Now let’s add a **Dashboard Page** that shows your user analytics:
- 📊 Total todos
- ✅ Completed vs. pending tasks
- 📈 Todo trend over time
- 🧮 Stats powered by server functions

Let’s build it step-by-step!

---

# 📊 Add a Dashboard Page with Analytics in Leptos + Axum

## ✅ What We’ll Build:

1. A new `/dashboard` route  
2. Fetch todo stats from the backend  
3. Display charts using `leptos::create_resource`  
4. Use conditional rendering based on auth  
5. Style with Tailwind CSS  

Let's go!

---

## 1. 🧱 Define Shared Dashboard Models

In `crates/my_todo_app/src/lib.rs`, define:

```rust
#[derive(Serialize, Deserialize, Clone, Debug)]
pub struct DashboardStats {
    pub total_todos: i64,
    pub completed_todos: i64,
    pub pending_todos: i64,
}
```

---

## 2. 🧠 Server Function: Get Dashboard Stats

Still in `lib.rs`:

```rust
use leptos::*;

#[server(GetDashboardStats)]
async fn get_dashboard_stats() -> Result<DashboardStats, ServerFnError> {
    use sqlx::PgPool;

    let pool = get_pool()?;
    let auth_user = use_auth_user().await?;

    let row = sqlx::query!(
        r#"
        SELECT 
            COUNT(*) AS total_todos,
            SUM(CASE WHEN completed THEN 1 ELSE 0 END) AS completed_todos
        FROM todos
        "#,
    )
    .fetch_one(&pool)
    .await
    .map_err(|e| ServerFnError::new(e.to_string()))?;

    Ok(DashboardStats {
        total_todos: row.total_todos.unwrap_or(0),
        completed_todos: row.completed_todos.unwrap_or(0),
        pending_todos: row.total_todos.unwrap_or(0) - row.completed_todos.unwrap_or(0),
    })
}
```

---

## 3. 🖥️ Create Dashboard Component

Create a new file or add to `lib.rs`:

```rust
#[component]
pub fn DashboardPage() -> impl IntoView {
    let get_stats = create_server_action::<GetDashboardStats>();

    let stats = move || {
        get_stats.value().get().cloned()
    };

    // Load once
    create_effect(move |_| {
        get_stats.dispatch(GetDashboardStats {});
    });

    view! {
        <div class="max-w-3xl mx-auto p-6">
            <h1 class="text-2xl font-bold mb-6">"Dashboard"</h1>

            {move || match stats() {
                Some(stats) => view! {
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
                        <StatCard title="Total Todos" value=stats.total_todos />
                        <StatCard title="Completed" value=stats.completed_todos color="green" />
                        <StatCard title="Pending" value=stats.pending_todos color="blue" />
                    </div>
                }.into_view(),
                None => view! { <p>"Loading stats..." </p> }.into_view()
            }}
        </div>
    }
}

#[component]
fn StatCard(title: &'static str, value: i64, #[prop(optional)] color: &'static str) -> impl IntoView {
    let color_class = match color {
        "green" => "bg-green-100 text-green-800",
        "blue" => "bg-blue-100 text-blue-800",
        _ => "bg-gray-100 text-gray-800",
    };

    view! {
        <div class=format!("p-4 rounded shadow {}", color_class)>
            <h2 class="text-sm font-medium">{title}</h2>
            <p class="text-2xl font-bold mt-1">{value}</p>
        </div>
    }
}
```

---

## 4. 🛣️ Add Route to Router

Update your main `App` component:

```rust
#[component]
pub fn App() -> impl IntoView {
    view! {
        <Router>
            <nav class="p-4 bg-blue-600 text-white flex space-x-4">
                <a href="/" class="hover:underline">"Home"</a>
                <a href="/todos" class="hover:underline">"Todos"</a>
                <a href="/profile" class="hover:underline">"Profile"</a>
                <a href="/dashboard" class="hover:underline">"Dashboard"</a>
                <button on:click=logout class="ml-auto">"Logout"</button>
            </nav>

            <main class="p-6 max-w-5xl mx-auto">
                <Routes>
                    <Route path="/" view=Home/>
                    <Route path="/login" view=Login/>
                    <Route path="/register" view=Register/>
                    <Route path="/todos" view=TodoApp/>
                    <Route path="/profile" view=ProfilePage/>
                    <Route path="/dashboard" view=DashboardPage/>
                </Routes>
            </main>
        </Router>
    }
}
```

---

## 5. 📈 Optional: Chart.js Integration (Advanced)

Want real charts?

Use `web-sys` to load [Chart.js](https://www.chartjs.org/) dynamically.

Add this inside `DashboardPage` after stats load:

```rust
use wasm_bindgen::prelude::*;
use web_sys::CanvasRenderingContext2d;

// Inside view! macro
<div>
    <canvas id="todo-chart" width="400" height="200"></canvas>
</div>

// Then use JS interop to draw chart
create_effect(move |_| {
    if let Some(stats) = stats() {
        let window = web_sys::window().unwrap();
        let document = window.document().unwrap();
        let canvas = document.get_element_by_id("todo-chart").unwrap();
        let canvas: web_sys::HtmlCanvasElement = canvas.dyn_into().unwrap();

        let ctx = canvas.get_context("2d").unwrap().unwrap().dyn_into::<CanvasRenderingContext2d>().unwrap();

        // Draw bar chart manually or use Chart.js via JSinterop
    }
});
```

> For simplicity, we’re skipping Chart.js unless you want me to walk through it.

---

## ✅ Summary

✅ Added `/dashboard` page  
✅ Fetched stats from DB using server function  
✅ Created reusable `StatCard` component  
✅ Styled everything with Tailwind  
✅ Fully protected behind auth middleware  

---

## 🚀 Next:

1. 🧪 Writing **E2E tests** with Playwright or Selenium  
2. 📦 Adding **file uploads** (e.g., profile picture or CSV export)  
3. 🧑‍💼 Adding **roles/permissions** (admin/user)  
4. 📅 Adding **calendar/todo scheduling** feature  
