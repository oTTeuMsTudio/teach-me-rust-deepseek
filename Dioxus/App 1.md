# Build XR and Mobile App using Dioxus

Dioxus is a powerful Rust-based framework for building user interfaces that can target multiple platforms, including web, desktop, mobile, and even virtual reality (XR). Writing VR/XR and mobile apps with Dioxus involves understanding how to structure your application, handle platform-specific features, and leverage the capabilities of Rust for performance and safety.

In this guide, we'll walk through the process of creating a simple VR/XR and mobile app using Dioxus. We'll cover:

1. **Setting Up Your Environment**
2. **Creating a Basic Dioxus App**
3. **Targeting Mobile Platforms**
4. **Adding VR/XR Support**
5. **Running and Testing Your App**

---

### 1. **Setting Up Your Environment**

Before you start, ensure you have the following installed:

- **Rust**: Install Rust via [rustup](https://rustup.rs/).
- **Cargo**: Comes bundled with Rust.
- **Node.js and npm**: Required for some Dioxus tooling.
- **Android SDK/NDK** (for mobile development): Set up Android Studio and configure the SDK/NDK paths.
- **Xcode** (for iOS development): Install Xcode from the Mac App Store.
- **VR/XR Tools**: Depending on your target platform (e.g., Oculus, WebXR), you may need specific SDKs or tools.

Install Dioxus CLI:
```bash
cargo install dioxus-cli
```

Verify the installation:
```bash
dx --version
```

---

### 2. **Creating a Basic Dioxus App**

Start by creating a new Dioxus project:
```bash
dx create my-vr-app
cd my-vr-app
```

This will generate a basic Dioxus project structure. Open the `src/main.rs` file to see the default app code:
```rust
use dioxus::prelude::*;

fn main() {
    launch(app);
}

fn app(cx: Scope) -> Element {
    cx.render(rsx! {
        div {
            h1 { "Hello, Dioxus!" }
            p { "Welcome to your first VR/XR app." }
        }
    })
}
```

Run the app in the browser to test it:
```bash
dx serve --web
```

Open `http://localhost:8080` in your browser to see the app.

---

### 3. **Targeting Mobile Platforms**

To target mobile platforms, you need to configure your project for Android and iOS.

#### For Android:
1. Add Android support to your `Cargo.toml`:
   ```toml
   [dependencies]
   dioxus = "0.3"
   dioxus-android = "0.3"
   ```

2. Build the Android app:
   ```bash
   dx build --android
   ```

3. Deploy to an Android device or emulator:
   ```bash
   dx deploy --android
   ```

#### For iOS:
1. Add iOS support to your `Cargo.toml`:
   ```toml
   [dependencies]
   dioxus = "0.3"
   dioxus-ios = "0.3"
   ```

2. Build the iOS app:
   ```bash
   dx build --ios
   ```

3. Open the generated Xcode project and run it on a simulator or device.

---

### 4. **Adding VR/XR Support**

To add VR/XR support, you can use WebXR (for browser-based VR) or platform-specific SDKs like Oculus or OpenXR.

#### Using WebXR:
WebXR is the easiest way to add VR support for browser-based apps. Update your `main.rs` to include WebXR components:
```rust
use dioxus::prelude::*;

fn main() {
    launch(app);
}

fn app(cx: Scope) -> Element {
    cx.render(rsx! {
        div {
            h1 { "VR Experience" }
            button {
                onclick: |_| {
                    // Trigger WebXR session
                    log::info!("Starting WebXR session...");
                },
                "Enter VR"
            }
        }
    })
}
```

Enable WebXR in your `index.html`:
```html
<script type="module">
    import 'https://cdn.jsdelivr.net/npm/@webxr-input-profiles/motion-controllers/dist/motion-controllers.module.js';
</script>
```

Run the app in a browser that supports WebXR (e.g., Chrome or Firefox):
```bash
dx serve --web
```

#### Using Platform-Specific SDKs:
For native VR/XR support, you can integrate libraries like OpenXR or Oculus SDK. This requires additional setup and Rust bindings. For example, you can use the `openxr` crate:
```toml
[dependencies]
openxr = "0.10"
```

Then, write platform-specific code to initialize and render VR content.

---

### 5. **Running and Testing Your App**

- **Web**: Use `dx serve --web` and test in a browser.
- **Mobile**: Use `dx deploy --android` or `dx deploy --ios` to test on devices.
- **VR/XR**: Test WebXR in supported browsers or use platform-specific tools for native VR.

---

### Example: Full VR/XR App

Here’s a complete example of a simple VR app using WebXR:
```rust
use dioxus::prelude::*;

fn main() {
    launch(app);
}

fn app(cx: Scope) -> Element {
    cx.render(rsx! {
        div {
            h1 { "VR Experience with Dioxus" }
            button {
                onclick: |_| {
                    // Simulate entering VR mode
                    log::info!("Entering VR mode...");
                },
                "Enter VR"
            }
        }
    })
}
```

---

### Tips for Success

1. **Modularize Your Code**: Separate platform-specific logic into modules for better maintainability.
2. **Optimize Performance**: Use Rust's zero-cost abstractions and efficient memory management to ensure smooth performance in VR/XR.
3. **Test Across Devices**: Test your app on various devices to ensure compatibility.
4. **Explore Dioxus Plugins**: Use plugins like `dioxus-router` for navigation and `dioxus-icons` for UI elements.
