# Leptos Website App

Creating a website with agents in Rust involves combining several concepts: asynchronous programming, web frameworks, and message-passing concurrency. Agents are typically used to handle tasks that need to run independently or in the background, such as fetching data from an API, managing state, or handling long-running operations.

Lets go through building a simple website using **Leptos** for the frontend (UI) and **Axum** with **Tokio** for the backend. We'll also use agents to manage background tasks.

---

### 1. Project Setup

First, set up your Rust project:

```bash
cargo new leptos_axum_website
cd leptos_axum_website
```

Add the necessary dependencies to your `Cargo.toml`:

```toml
[dependencies]
leptos = "0.4" # Use the latest version
axum = "0.7"
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tower-http = "0.3" # For CORS support
```

---

### 2. Backend with Axum and Tokio

Create a file named `src/main.rs` for the backend server:

```rust
use axum::{
    routing::get,
    Router,
};
use tower_http::cors::{CorsLayer, AllowOrigin};
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    // Create a channel for communication between the agent and the server
    let (tx, mut rx) = mpsc::channel::<String>(10);

    // Spawn an agent task
    tokio::spawn(async move {
        while let Some(message) = rx.recv().await {
            println!("Agent received: {}", message);
            // Simulate some background work
            tokio::time::sleep(std::time::Duration::from_secs(2)).await;
            println!("Agent processed: {}", message);
        }
    });

    // Define a route that sends a message to the agent
    let app = Router::new()
        .route("/", get(|| async { "Hello from Axum!" }))
        .route("/send_message", get(move |message: String| {
            let tx_clone = tx.clone();
            async move {
                tx_clone.send(message).await.unwrap();
                "Message sent to agent!"
            }
        }))
        .layer(
            CorsLayer::new()
                .allow_origin(AllowOrigin::any())
                .allow_methods(["GET", "POST"]),
        );

    let addr = "127.0.0.1:3000".parse().unwrap();
    println!("Server running on http://{}", addr);
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

Here:
- We define an Axum server with two routes: `/` and `/send_message`.
- A Tokio `mpsc` channel is used to communicate with the agent.
- The agent processes messages in the background.

---

### 3. Frontend with Leptos

Create a file named `src/lib.rs` for the Leptos frontend:

```rust
use leptos::*;

#[component]
fn App(cx: Scope) -> impl IntoView {
    let (input_value, set_input_value) = create_signal(cx, String::new());
    let (response, set_response) = create_signal(cx, String::new());

    let send_message = move |_| {
        spawn_local(async move {
            let client = reqwest::Client::new();
            let res = client
                .get(format!("http://127.0.0.1:3000/send_message?message={}", input_value.get()))
                .send()
                .await
                .unwrap()
                .text()
                .await
                .unwrap();
            set_response.set(res);
        });
    };

    view! { cx,
        <h1>"Leptos Agent Example"</h1>
        <input
            type="text"
            value=input_value
            on:input=move |ev| set_input_value.set(event_target_value(&ev))
        />
        <button on:click=send_message>"Send Message"</button>
        <p>{response}</p>
    }
}

#[cfg(feature = "ssr")]
#[tokio::main]
async fn main() {
    use axum::{Router, routing::get};
    use leptos::*;
    use leptos_axum::{generate_route_list, LeptosRoutes};

    let conf = get_configuration(None).await.unwrap();
    let addr = format!("127.0.0.1:8080").parse().unwrap();

    let routes = generate_route_list(App);

    let app = Router::new()
        .route("/api/*fn_name", get(leptos_axum::handle_server_fns))
        .leptos_routes(conf.leptos_options, routes, App)
        .fallback(file_and_error_handler);

    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

#[cfg(not(feature = "ssr"))]
fn main() {
    console_error_panic_hook::set_once();
    leptos::mount_to_body(|cx| view! { cx, <App/> });
}

async fn file_and_error_handler(uri: axum::http::Uri) -> Result<impl axum::response::IntoResponse, (axum::http::StatusCode, String)> {
    let path = uri.path().trim_start_matches('/').to_string();
    if let Ok(file) = tokio::fs::read(path).await {
        return Ok(axum::response::Response::builder()
            .header("content-type", mime_guess::from_path(&path).first_or_octet_stream().as_ref())
            .body(file)
            .unwrap());
    }
    Err((axum::http::StatusCode::NOT_FOUND, "Not Found".to_string()))
}
```

Here:
- We define a simple UI with an input field and a button.
- When the button is clicked, it sends a message to the backend via an HTTP request.
- The response from the backend is displayed on the page.

---

### 4. Running the Application

To run the application:
1. Start the backend server:
   ```bash
   cargo run --bin leptos_axum_website
   ```
2. Open your browser and navigate to `http://127.0.0.1:8080`.

---

### 5. Explanation of Agents

In this example:
- The backend uses a Tokio `mpsc` channel to simulate an agent that processes messages in the background.
- The frontend communicates with the backend using HTTP requests.
- This setup demonstrates how agents can be used to offload work from the main thread and handle tasks asynchronously.

You can extend this pattern to include more complex agents, such as those that fetch data from external APIs, manage WebSocket connections, or handle user authentication.
