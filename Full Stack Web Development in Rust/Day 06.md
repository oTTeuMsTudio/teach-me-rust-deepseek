# Building a full-stack, real-time, authenticated Rust web app** with Leptos

Now let’s add **AI integration** to make your todo app even smarter.

---

# 🤖 Add AI Features with OpenAI (e.g., Auto-Suggest or Smart Sorting)

We’ll integrate **OpenAI's API** into your Leptos + Axum app to:
- ✅ Suggest smart titles for new todos
- ✅ Automatically sort or categorize tasks
- ✅ Use server functions securely on the backend

> We'll use `reqwest` to call OpenAI APIs from the Rust backend — never exposing the API key to the client.

---

## 1. 🛠️ Setup Dependencies

Add these to your `my_todo_app_axum/Cargo.toml`:

```toml
reqwest = { version = "0.11", features = ["json"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

---

## 2. 🔐 Set OpenAI API Key in `.env`

In `.env` file:

```env
OPENAI_API_KEY=your-openai-api-key-here
```

> Get the key from: [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)

---

## 3. 🧠 Define AI Request/Response Types

Create a new file: `src/ai.rs` inside `my_todo_app_axum`

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
pub struct OpenAiRequest {
    pub model: String,
    pub prompt: String,
    pub max_tokens: u32,
}

#[derive(Serialize, Deserialize)]
pub struct OpenAiResponse {
    pub choices: Vec<Choice>,
}

#[derive(Serialize, Deserialize)]
pub struct Choice {
    pub text: String,
}
```

---

## 4. 🧠 Server Function: AI Todo Suggestion

Back in `crates/my_todo_app/src/lib.rs`, define a new server function:

```rust
use leptos::*;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
pub struct AiSuggestion {
    pub suggestion: String,
}

#[server(GetTodoSuggestion)]
async fn get_todo_suggestion(prompt: String) -> Result<AiSuggestion, ServerFnError> {
    use crate::ai::{OpenAiRequest, OpenAiResponse};

    let client = reqwest::Client::new();
    let api_key = std::env::var("OPENAI_API_KEY").map_err(|_| ServerFnError::new("Missing OPENAI_API_KEY"))?;

    let request = OpenAiRequest {
        model: "text-davinci-003".to_string(),
        prompt: format!("Suggest a concise todo title based on: {}", prompt),
        max_tokens: 60,
    };

    let response = client
        .post("https://api.openai.com/v1/completions")
        .header("Authorization", format!("Bearer {}", api_key))
        .json(&request)
        .send()
        .await
        .map_err(|e| ServerFnError::new(e.to_string()))?
        .json::<OpenAiResponse>()
        .await
        .map_err(|e| ServerFnError::new(e.to_string()))?;

    Ok(AiSuggestion {
        suggestion: response.choices[0].text.trim().to_string(),
    })
}
```

---

## 5. 🖥️ Frontend: Show AI Suggestion Box

Update your frontend component (`lib.rs`) to show an AI suggestion field:

```rust
#[component]
pub fn TodoApp() -> impl IntoView {
    let suggest_action = create_server_action::<GetTodoSuggestion>();
    let input_ref = create_node_ref::<Input>();
    
    let on_input = move |ev| {
        let prompt = event_target_value(&ev);
        if prompt.len() > 3 {
            suggest_action.dispatch(GetTodoSuggestion { prompt });
        }
    };

    view! {
        <div>
            <h2>"Smart Todo Input"</h2>
            <input type="text" 
                   node_ref=input_ref
                   on:input=on_input 
                   placeholder="Type task idea..." />
            
            {move || suggest_action.value().get()
                .flatten()
                .map(|s| view! {
                    <p><b>"AI Suggestion:"</b> {s.suggestion}</p>
                })
            }

            // Your existing todo list here...
        </div>
    }
}
```

Now as the user types, the AI suggests better-formatted todo titles!

---

## 6. 🎯 Bonus Ideas for AI Integration

next:

### ✅ Smart Task Prioritization
Ask GPT to return a priority score or category.

Prompt:
```txt
Classify this task into: Urgent, Important, Low Priority
Task: Buy groceries
```

### ✅ Task Categorization
Group tasks by theme like “Work”, “Personal”, “Errands”.

### ✅ Natural Language Parsing
Let users type free-form sentences and extract actionable tasks.

Example:
> “I should reply to John about the meeting tomorrow at 2pm”

→ Extracted task: `"Reply to John about meeting"`

---

## 🚀 Next:

1. 📱 Making your app mobile-responsive with **Tailwind CSS**
2. 🧪 Writing **unit tests** for server functions and AI logic
3. 📊 Adding **analytics or logging** using tracing
4. 🧩 Creating a **dashboard or profile page**
