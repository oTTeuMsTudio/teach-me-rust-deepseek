# Form input and full CRUD functionality for Leptos + Axum To-Do app.

Add:
- ✅ A form to **add new todos**
- ✅ A backend route to **insert into the database**
- ✅ Re-render the list after adding
- ✅ (Bonus) Add a way to **delete todos**

---

## 🧱 1. Update `src/lib.rs` — Add Form Input

Update the `TodoApp` component to include a form:

```rust
use leptos::*;

#[component]
fn TodoApp() -> impl IntoView {
    let fetch_todos = create_resource(|| (), |_| async move {
        reqwasm::http::Request::get("/api/todos")
            .send()
            .await
            .unwrap()
            .json::<Vec<String>>()
            .await
            .unwrap()
    });

    // Form state
    let (input, set_input) = create_signal("".to_string());

    // Submit handler
    let submit_todo = move |_| {
        let todo_text = input();
        spawn_local(async move {
            reqwasm::http::Request::post("/api/todos")
                .header("Content-Type", "application/json")
                .body(serde_json::to_string(&todo_text).unwrap())
                .send()
                .await
                .unwrap();
        });
        set_input("".to_string());
    };

    view! {
        <h2>"To-Do List"</h2>
        <form on:submit=submit_todo>
            <input
                type="text"
                placeholder="Add a new task..."
                prop:value=input
                on:input=move |ev| set_input(event_target_value(&ev))
            />
            <button type="submit">"Add"</button>
        </form>

        <Suspense fallback=|| ()>
            {move || {
                fetch_todos.get().map(|todos| {
                    view! {
                        <ul class="todo-list">
                            {todos.into_iter().map(|t| view!{<li>{t}</li>}).collect::<Vec<_>>()}
                        </ul>
                    }
                })
            }}
        </Suspense>
    }
}
```

---

## 🌐 2. Update `src/main.rs` — Add POST Route for New Todos

Add function:

```rust
async fn add_todo(
    State(state): State<Arc<AppState>>,
    axum::extract::Json(payload): axum::extract::Json<serde_json::Value>,
) -> &'static str {
    if let Some(text) = payload.get("text").and_then(|v| v.as_str()) {
        sqlx::query!("INSERT INTO todos (text) VALUES (?)", text)
            .execute(&state.pool)
            .await
            .unwrap();
    }
    "OK"
}
```

Then update your router in `main()`:

```rust
let app = Router::new()
    .route("/", get(move || {
        let options = leptos_options.clone();
        move || generate_route(App, &options)
    }))
    .route("/api/todos", get(get_todos).post(add_todo))
    .with_state(Arc::clone(&state));
```

---

## 💾 3. Optional: Add Delete Functionality

### a. Backend (in `main.rs`)

Add a DELETE route:

```rust
async fn delete_all_todos(State(state): State<Arc<AppState>>) -> &'static str {
    sqlx::query!("DELETE FROM todos")
        .execute(&state.pool)
        .await
        .unwrap();
    "All todos deleted"
}
```

And route it:

```rust
.route("/api/todos", get(get_todos).post(add_todo).delete(delete_all_todos))
```

### b. Frontend (in `lib.rs`)

Add a delete button inside the `<Suspense>` block:

```rust
<button on:click=move |_| {
    spawn_local(async {
        reqwasm::http::Request::delete("/api/todos")
            .send()
            .await
            .unwrap();
    });
}>Delete All</button>
```

---

## 📦 4. Commit Changes & Push to GitHub

```bash
git add .
git commit -m "Add form input and CRUD routes for To-Do app"
git push origin main
```

---

## 🚀 Test It Out

Run the dev server again:

```bash
trunk serve --port 3000
```

Open [http://localhost:3000](http://localhost:3000), try adding and deleting todos!

---

## 🎉 Full-Stack Rust App with Leptos Done.

🔧 Highlights:
- Written entirely in Rust
- No JavaScript
- Interactive UI using Leptos
- REST API via Axum
- SQLite persistence
- Ready to deploy or expand
