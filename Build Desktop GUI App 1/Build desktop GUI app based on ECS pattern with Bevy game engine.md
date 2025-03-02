https://muongkimhong.github.io/famiq/gettingstart.html
https://bevyengine.org/learn/quick-start/introduction/
https://github.com/MuongKimhong/famiq?tab=readme-ov-file

# Build desktop GUI app based on ECS pattern with Bevy game engine

Building a desktop GUI application with Rust is an exciting endeavor, and the Famiq framework (linked in your question) seems to be a promising tool for this purpose. Lets build a desktop GUI app using Bevy.

---

### **1. Prerequisites**
Before diving into the development process, ensure you have the following installed:
- **Rust**: Install Rust by following the instructions on [rust-lang.org](https://www.rust-lang.org/tools/install).
- **Cargo**: This comes bundled with Rust and is used to manage dependencies and build projects.
- **Famiq**: Familiarize yourself with the Famiq framework by reading its documentation at [https://muongkimhong.github.io/famiq/](https://muongkimhong.github.io/famiq/).

---

### **2. Setting Up Your Project**
To start, create a new Rust project using Cargo:

```bash
cargo new famiq_gui_app
cd famiq_gui_app
```

This creates a basic Rust project structure. Next, add the Famiq dependency to your `Cargo.toml` file:

```toml
[dependencies]
famiq = "0.3" # Replace with the latest version if available
```

Run `cargo build` to download and compile the dependencies.

---

### **3. Understanding Famiq Basics**
Famiq is a reactive GUI framework inspired by Elm. It uses a model-view-update architecture:
- **Model**: Represents the state of your application.
- **View**: Defines how the UI looks and behaves.
- **Update**: Handles user interactions and updates the model.

The core components of a Famiq app are:
- `App`: The main entry point for your application.
- `Component`: A reusable piece of UI logic.
- `Msg`: Messages that trigger updates in the model.

---

### **4. Building a Simple GUI App**
Let’s create a simple counter application where users can increment or decrement a number.

#### **Step 1: Define the Model**
In `src/main.rs`, define the model to store the counter value:

```rust
#[derive(Debug, Clone)]
struct Model {
    count: i32,
}

impl Default for Model {
    fn default() -> Self {
        Model { count: 0 }
    }
}
```

Here, `count` represents the current value of the counter.

---

#### **Step 2: Define Messages**
Define the messages that will update the model:

```rust
#[derive(Debug, Clone)]
enum Msg {
    Increment,
    Decrement,
}
```

These messages represent user actions like clicking the increment or decrement buttons.

---

#### **Step 3: Implement the Update Function**
Create the `update` function to handle messages and update the model:

```rust
fn update(msg: Msg, model: &mut Model) {
    match msg {
        Msg::Increment => model.count += 1,
        Msg::Decrement => model.count -= 1,
    }
}
```

This function modifies the `count` field in the model based on the message received.

---

#### **Step 4: Define the View**
Use Famiq's API to define the UI:

```rust
use famiq::prelude::*;

fn view(model: &Model) -> Node<Msg> {
    div(
        vec![],
        vec![
            button(vec![on_click(|_| Msg::Increment)], vec![text("Increment")]),
            text(&model.count.to_string()),
            button(vec![on_click(|_| Msg::Decrement)], vec![text("Decrement")]),
        ],
    )
}
```

Here:
- `div` creates a container for the UI elements.
- `button` creates clickable buttons.
- `on_click` attaches event handlers to buttons.
- `text` displays plain text.

---

#### **Step 5: Initialize the App**
Finally, initialize the Famiq app in the `main` function:

```rust
fn main() {
    let model = Model::default();
    famiq::start_app(
        "app-container", // ID of the HTML element where the app will render
        model,
        view,
        update,
    );
}
```

This starts the Famiq app, rendering it inside an HTML element with the ID `app-container`.

---

### **5. Running the App**
To run the app, you need to serve it in a web browser. Create an `index.html` file in the `famiq_gui_app` directory:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Famiq Counter App</title>
</head>
<body>
    <div id="app-container"></div>
    <script type="module">
        import init from './target/wasm32-unknown-unknown/release/famiq_gui_app.js';
        init();
    </script>
</body>
</html>
```

Build the app with WebAssembly support:

```bash
cargo install trunk
trunk build --release
```

Serve the app locally:

```bash
trunk serve
```

Open your browser and navigate to `http://localhost:8080` to see the app in action.

---

### **6. Key Takeaways**
- Famiq simplifies GUI development by enforcing a clean model-view-update architecture.
- Rust's safety and performance make it an excellent choice for building robust desktop applications.
- By combining Rust with Famiq, you can create efficient, reactive GUIs for both desktop and web platforms.
