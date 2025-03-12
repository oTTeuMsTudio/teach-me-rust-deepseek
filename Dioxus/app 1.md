Dioxus is a modern, declarative UI framework for Rust that allows you to build websites, mobile apps, and desktop applications using a single codebase. It is inspired by React and Flutter, making it familiar to developers who have experience with these frameworks. Lets get started with Dioxus to build a website and a mobile app.

---

### **Step 1: Setting Up Your Environment**

Before diving into coding, ensure your development environment is ready:

1. **Install Rust**:
   - If you don't already have Rust installed, download it from [rust-lang.org](https://www.rust-lang.org/tools/install) and follow the installation instructions.
   - Verify the installation by running:
     ```bash
     rustc --version
     ```

2. **Install Dioxus CLI**:
   - The Dioxus CLI simplifies project creation and management. Install it using Cargo:
     ```bash
     cargo install dioxus-cli
     ```
   - Verify the installation:
     ```bash
     dx --version
     ```

3. **Set Up for Mobile Development (Optional)**:
   - For mobile app development, you'll need to set up the Android SDK and NDK. Follow the official [Dioxus mobile setup guide](https://dioxuslabs.com/guide/mobile/setup.html) for detailed instructions.

---

### **Step 2: Creating a New Dioxus Project**

1. **Create a New Project**:
   Use the Dioxus CLI to scaffold a new project:
   ```bash
   dx create my-dioxus-app
   cd my-dioxus-app
   ```

2. **Project Structure**:
   After creating the project, you'll see a structure like this:
   ```
   my-dioxus-app/
   ├── src/
   │   └── main.rs       # Entry point for your app
   ├── Cargo.toml        # Rust project configuration
   └── dioxus.toml       # Dioxus-specific configuration
   ```

3. **Run the App in Web Mode**:
   To test the app as a website, run:
   ```bash
   dx serve
   ```
   Open your browser at `http://localhost:8080` to see the default app.

---

### **Step 3: Writing Your First Dioxus Component**

Dioxus uses a declarative syntax similar to React or Flutter. Let's create a simple counter app.

1. **Edit `src/main.rs`**:
   Replace the contents of `main.rs` with the following code:

   ```rust
   use dioxus::prelude::*;

   fn main() {
       dioxus::web::launch(app);
   }

   fn app(cx: Scope) -> Element {
       let count = use_state(&cx, || 0);

       cx.render(rsx! {
           div {
               h1 { "Counter App" }
               p { "Count: {count}" }
               button {
                   onclick: move |_| count += 1,
                   "Increment"
               }
               button {
                   onclick: move |_| count -= 1,
                   "Decrement"
               }
           }
       })
   }
   ```

2. **Explanation**:
   - `use_state`: A hook to manage state (similar to React's `useState`).
   - `rsx!`: A macro for writing JSX-like UI templates.
   - `onclick`: Event handler for button clicks.

3. **Run the App**:
   Start the development server again:
   ```bash
   dx serve
   ```
   You should see a counter app in your browser.

---

### **Step 4: Building for Mobile**

To adapt your app for mobile, follow these steps:

1. **Update `dioxus.toml`**:
   Ensure your `dioxus.toml` includes the mobile target:
   ```toml
   [application]
   name = "my-dioxus-app"
   default_platform = "mobile"

   [platform.mobile]
   enabled = true
   ```

2. **Build for Android**:
   Run the following command to build the app for Android:
   ```bash
   dx build --platform android
   ```

3. **Run on an Emulator or Device**:
   Use the Android emulator or connect a physical device to test your app:
   ```bash
   dx serve --platform android
   ```

4. **iOS Support**:
   For iOS, you'll need macOS and Xcode. Follow the [Dioxus iOS setup guide](https://dioxuslabs.com/guide/mobile/ios.html) for details.

---

### **Step 5: Styling Your App**

Dioxus supports CSS for styling. Here's how to add styles:

1. **Inline Styles**:
   Add inline styles directly in your `rsx!` block:
   ```rust
   rsx! {
       div {
           style: "background-color: lightblue; padding: 20px;",
           h1 { "Styled Counter App" }
       }
   }
   ```

2. **External CSS**:
   Create a `styles.css` file in your project directory and link it in your app:
   ```rust
   rsx! {
       link { rel: "stylesheet", href: "styles.css" }
   }
   ```

---

### **Step 6: Deploying Your App**

1. **Web Deployment**:
   Build the app for production:
   ```bash
   dx build --release --platform web
   ```
   Deploy the generated files in the `dist` folder to your hosting provider (e.g., Netlify, Vercel).

2. **Mobile Deployment**:
   - For Android, generate an APK:
     ```bash
     dx build --release --platform android
     ```
   - For iOS, follow Apple's guidelines for App Store deployment.

---

### **Conclusion**

You now have a basic understanding of how to build a website and mobile app using Dioxus in Rust. Here's a quick recap of what we covered:

1. Setting up your environment with Rust and Dioxus CLI.
2. Creating and running a Dioxus project.
3. Writing a simple counter app using Dioxus components.
4. Adapting the app for mobile platforms.
5. Styling your app with CSS.
6. Deploying your app for web and mobile.

From here, you can explore advanced features like routing, state management, and integrating APIs. The [Dioxus documentation](https://dioxuslabs.com/guide/) is an excellent resource for further learning.
