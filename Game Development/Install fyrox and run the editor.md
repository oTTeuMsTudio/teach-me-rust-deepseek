# Fyrox The Rust Game Engine
Fyrox (also known as Fyrox Engine) is a modern, feature-rich, and data-driven game engine written in Rust. It provides tools for creating 2D and 3D games, simulations, and other interactive applications. The Fyrox editor is a powerful tool that allows you to create and manage scenes, assets, and other resources visually.

Below is a step-by-step guide to installing Fyrox and running the editor on your system.

---

### **Step 1: Prerequisites**
Before installing Fyrox, ensure you have the following prerequisites installed:

1. **Rust Toolchain**:
   - Install Rust by following the official instructions: [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install).
   - After installation, verify it by running:
     ```bash
     rustc --version
     cargo --version
     ```
   - Ensure you have the latest stable version of Rust. You can update it using:
     ```bash
     rustup update
     ```

2. **Git**:
   - Install Git if it’s not already installed. You can download it from [https://git-scm.com/](https://git-scm.com/).

3. **C++ Build Tools**:
   - On Windows, install the Visual Studio Build Tools (C++ workload).
   - On Linux, install `build-essential` or equivalent development tools.
   - On macOS, install Xcode Command Line Tools:
     ```bash
     xcode-select --install
     ```

4. **Graphics Drivers**:
   - Ensure your graphics drivers are up-to-date, as Fyrox relies on Vulkan for rendering.

---

### **Step 2: Clone the Fyrox Repository**
Fyrox is hosted on GitHub. Clone the repository to your local machine:

```bash
git clone https://github.com/FyroxEngine/Fyrox.git
cd Fyrox
```

---

### **Step 3: Build the Editor**
The Fyrox editor is part of the repository and needs to be built using Cargo. Follow these steps:

1. **Install Dependencies**:
   - Run the following command to install the required dependencies:
     ```bash
     cargo build --features editor
     ```

2. **Build the Editor**:
   - To build the editor in release mode (recommended for better performance), use:
     ```bash
     cargo build --release --features editor
     ```

   - This will generate the editor binary in the `target/release` directory.

---

### **Step 4: Run the Editor**
Once the editor is built, you can run it as follows:

1. Navigate to the `target/release` directory:
   ```bash
   cd target/release
   ```

2. Run the editor:
   ```bash
   ./fyroxed
   ```

   - On Windows, the executable will be named `fyroxed.exe`.

---

### **Step 5: Explore the Editor**
When the editor launches, you’ll see a graphical interface where you can:

- Create and edit scenes.
- Manage assets (textures, models, etc.).
- Configure game objects and their properties.
- Preview your work in real-time.

Refer to the official documentation for detailed usage instructions: [https://fyrox.rs/docs/](https://fyrox.rs/docs/).

---

### **Troubleshooting**
If you encounter any issues during installation or execution, consider the following:

1. **Outdated Dependencies**:
   - Ensure all dependencies are up-to-date by running:
     ```bash
     cargo update
     ```

2. **Graphics Issues**:
   - If the editor fails to launch due to graphics-related errors, verify your Vulkan drivers are installed correctly.

3. **Missing Features**:
   - Ensure you’re building with the `editor` feature flag enabled.

4. **Check Logs**:
   - Look at the terminal output or logs for error messages and address them accordingly.

---

### **Additional Notes**
- **Custom Projects**: If you want to integrate Fyrox into your own project, add it as a dependency in your `Cargo.toml` file:
  ```toml
  [dependencies]
  fyrox = "0.27" # Replace with the latest version
  ```
  Then, use the provided APIs to build your game or application.

- **Community Support**: Join the Fyrox community on Discord or GitHub Discussions for help and updates.

---

By following these steps, you should be able to install Fyrox and run its editor successfully. Let me know if you encounter any specific issues or need further clarification!
