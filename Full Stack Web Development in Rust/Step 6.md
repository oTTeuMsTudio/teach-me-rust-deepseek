# Next level. Add Tailwind CSS to style full-stack Leptos To-Do app and make it look modern, responsive, and beautiful — all while staying in Rust!

---

## 🎨 Why Add Tailwind?

Tailwind gives you:
- Utility-first CSS (no need for custom stylesheets)
- Responsive design
- Easy integration with HTML templates
- Speedy development

We’ll use the **CDN version** of Tailwind so you don’t have to set up a build system like PostCSS or Vite.

---

# 🧱 Step 1: Update `index.html` to Use Tailwind CDN

Replace your current `index.html` with this:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Leptos To-Do App</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center p-6">
  <div id="root" class="w-full max-w-md bg-white shadow-lg rounded-lg p-6"></div>
</body>
</html>
```

This sets up:
- Tailwind via CDN
- Centered layout
- White card container
- Responsive padding and fonts

---

# 🖥️ Step 2: Style the To-Do List in `src/lib.rs`

Update the `TodoApp` component to use Tailwind classes.

Here’s the updated version:

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

    // Delete handler
    let delete_todos = move |_| {
        spawn_local(async move {
            reqwasm::http::Request::delete("/api/todos")
                .send()
                .await
                .unwrap();
        });
    };

    view! {
        <div class="space-y-6">
            <h2 class="text-2xl font-bold text-gray-700">"To-Do List"</h2>

            <form on:submit=submit_todo class="flex gap-2">
                <input
                    type="text"
                    placeholder="Add a new task..."
                    prop:value=input
                    on:input=move |ev| set_input(event_target_value(&ev))
                    class="flex-grow px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                />
                <button
                    type="submit"
                    class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition"
                >
                    "Add"
                </button>
            </form>

            <button
                on:click=delete_todos
                class="text-sm text-red-500 hover:text-red-700 underline"
            >
                "Delete All"
            </button>

            <Suspense fallback=|| ()>
                {move || {
                    fetch_todos.get().map(|todos| {
                        view! {
                            <ul class="space-y-2">
                                {todos.into_iter().map(|t| view!{
                                    <li class="p-3 bg-gray-50 border border-gray-200 rounded-md shadow-sm text-gray-800">
                                        {t}
                                    </li>
                                }).collect::<Vec<_>>()}
                            </ul>
                        }
                    })
                }}
            </Suspense>
        </div>
    }
}
```

---

## ✅ Step 3: Rebuild the App

Make sure everything is working:

```bash
trunk serve --port 3000
```

Open [http://localhost:3000](http://localhost:3000)

You should now see a beautifully styled To-Do list that looks clean and modern ✅

---

## 📦 Step 4: Commit and Push Changes

```bash
git add .
git commit -m "Add Tailwind CSS styling to To-Do app"
git push origin main
```

---

## 🚀 App Is Now Styled

- Added **Tailwind CSS via CDN**
- Styled your form and todo list
- Made it mobile-friendly and responsive
- Kept everything in Rust + Leptos

---

## 🌟 Next:
1. Add **user authentication (JWT)**?
2. Set up **CI/CD with GitHub Actions**?
3. Add **UUIDs or timestamps** to todos?
4. Deploy to **Cloudflare Pages** or **Vercel**?

Just say what you'd like next 👇 I’m here to help you build a production-ready full-stack Rust app!
