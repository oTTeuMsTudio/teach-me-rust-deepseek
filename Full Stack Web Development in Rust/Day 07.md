We're building a full-stack Rust app with **Leptos**, and we've already added:
- ✅ PostgreSQL database
- ✅ Authentication (JWT)
- ✅ Real-time updates via WebSockets
- ✅ AI-powered todo suggestions

Now let’s make it **mobile-friendly** using **Tailwind CSS** — the utility-first CSS framework that makes responsive design easy.

---

# 📱 Make Your Leptos App Mobile-Friendly with Tailwind CSS

## ✅ What We’ll Do:

1. Add **Tailwind CSS** to your Leptos project  
2. Set up **responsive utilities**  
3. Style your todo list for desktop & mobile  
4. Use **flexbox/grid** for layout  
5. Add **dark mode toggle** (optional bonus)

Let's go!

---

## 1. 🧰 Install Tailwind CSS

Since we’re using `trunk` as the build tool, we'll use a CDN version of Tailwind for simplicity (no extra build steps needed).

> For production, you can later switch to JIT or PostCSS.

Update  `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Leptos Todo App</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <!-- Tailwind CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 text-gray-900">
  <div id="main" class="min-h-screen flex items-center justify-center p-4"></div>
  <script type="module" src="/main.js"></script>
</body>
</html>
```

---

## 2. 🎨 Style Components with Tailwind

### Example: Responsive Login Form (`Login` component)

```rust
#[component]
pub fn Login() -> impl IntoView {
    let login_action = create_server_action::<Login>();

    view! {
        <div class="max-w-md w-full bg-white p-6 rounded shadow-md mx-auto">
            <h2 class="text-2xl font-bold mb-4">"Login"</h2>
            <ActionForm action=login_action class="space-y-4">
                <input
                    type="text"
                    name="email"
                    placeholder="Email"
                    class="w-full px-3 py-2 border rounded focus:outline-none focus:ring focus:border-blue-400"
                />
                <input
                    type="password"
                    name="password"
                    placeholder="Password"
                    class="w-full px-3 py-2 border rounded focus:outline-none focus:ring focus:border-blue-400"
                />
                <button
                    type="submit"
                    class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded transition-colors"
                >
                    "Login"
                </button>
            </ActionForm>

            {move || login_action.value().get()
                .flatten()
                .map(|token| view! {
                    <p class="mt-4 text-green-600">"Logged in!"</p>
                })
            }
        </div>
    }
}
```

This looks great on both mobile and desktop!

---

## 3. 🧱 Responsive Todo List UI

Update your main `TodoApp` view:

```rust
#[component]
pub fn TodoApp() -> impl IntoView {
    let todos = create_resource(|| (), |_| async move {
        get_todos().await.unwrap_or_default()
    });

    view! {
        <div class="max-w-lg mx-auto p-4">
            <h1 class="text-3xl font-bold mb-4">"Your Todos"</h1>

            <form
                on:submit=move |ev| {
                    ev.prevent_default();
                    let title = event_target_value(&ev, "title");
                    create_todo_action.dispatch(CreateTodo { title });
                }
                class="flex gap-2 mb-6"
            >
                <input
                    type="text"
                    name="title"
                    placeholder="New task..."
                    class="flex-grow px-3 py-2 border rounded focus:outline-none focus:ring focus:border-blue-400"
                />
                <button
                    type="submit"
                    class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded"
                >
                    "Add"
                </button>
            </form>

            <ul class="space-y-2">
                {move || todos.get().unwrap_or_default().into_iter().map(|todo| {
                    view! {
                        <li class="bg-white p-3 rounded shadow-sm flex items-center justify-between">
                            <label class="flex items-center gap-2">
                                <input
                                    type="checkbox"
                                    checked=todo.completed
                                    class="h-4 w-4"
                                    on:click=move |_| {
                                        toggle_todo_action.dispatch(ToggleTodo { id: todo.id })
                                    }
                                />
                                <span class=if todo.completed {
                                    "line-through text-gray-500"
                                } else {
                                    ""
                                }>
                                    {todo.title}
                                </span>
                            </label>
                            <button
                                on:click=move |_| {
                                    delete_todo_action.dispatch(DeleteTodo { id: todo.id })
                                }
                                class="text-red-500 hover:text-red-700 text-sm"
                            >
                                "X"
                            </button>
                        </li>
                    }
                }).collect::<Vec<_>>()}
            </ul>
        </div>
    }
}
```

---

## 4. 💡 Bonus: Add Dark Mode Toggle

Want to impress users with dark mode? Here’s how:

Add this button somewhere:

```rust
<button
    on:click=move |_| {
        document.body().unwrap().class_list().toggle("dark").unwrap();
    }
    class="mt-4 px-4 py-2 bg-gray-800 text-white rounded"
>
    "Toggle Dark Mode"
</button>
```

Then add some global styles in `index.html`:

```html
<style>
  body.dark {
    @apply bg-gray-900 text-white;
  }
  body.dark input, body.dark button {
    @apply bg-gray-800 text-white border-gray-600;
  }
</style>
```

---

## ✅ Summary

✅ We now have a **fully responsive Leptos app** styled with **Tailwind CSS**  
✅ Works beautifully on desktop and mobile  
✅ Used utility-first classes for speed and flexibility  
✅ Added optional dark mode toggle  

---

## 🚀 Next:

1. 🧪 Writing **unit tests** for server functions and components  
2. 📊 Adding **analytics or logging** with tracing  
3. 🔐 Securing headers with **Axum middleware**  
4. 🧩 Creating a **dashboard or profile page**
