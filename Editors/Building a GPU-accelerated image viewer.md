# Building a GPU-accelerated image viewer
Building a GPU-accelerated image viewer using **Iced** and **wgpu** in Rust is an excellent way to explore modern GUI development and GPU programming. Lets go through the process, breaking it into manageable parts. We are learning a basic of how to integrate these libraries to create a functional image viewer.

---

### **1. Overview of Components**
- **Iced**: A cross-platform GUI library for Rust that provides a simple and declarative API for building user interfaces.
- **wgpu**: A low-level graphics API that allows you to interact with the GPU directly, enabling hardware-accelerated rendering.
- **Image Viewer**: The application will load an image file, render it using wgpu, and display it in an Iced window.

---

### **2. Prerequisites**
Before starting, ensure you have the following:
- **Rust installed**: Use `rustup` to install Rust if you haven't already.
- **Cargo**: Rust's package manager and build system.
- **Basic knowledge of Rust**: Familiarity with Rust syntax and concepts like ownership, lifetimes, and traits.
- **Graphics drivers**: Ensure your system has up-to-date GPU drivers compatible with wgpu (Vulkan, Metal, or DirectX).

---

### **3. Project Setup**
1. **Create a new Rust project**:
   ```bash
   cargo new gpu_image_viewer
   cd gpu_image_viewer
   ```

2. **Add dependencies**:
   Edit `Cargo.toml` to include the required crates:
   ```toml
   [dependencies]
   iced = "0.10" # Check for the latest version
   iced_wgpu = "0.10" # Integration between Iced and wgpu
   wgpu = "0.15" # Check for the latest version
   image = "0.24" # For loading image files
   ```

3. **Initialize the project**:
   Run `cargo build` to download and compile the dependencies.

---

### **4. Application Structure**
The application will consist of the following components:
1. **Main Application Logic**: Handles the Iced GUI setup and event loop.
2. **Image Loading**: Loads an image file using the `image` crate.
3. **GPU Rendering**: Uses wgpu to render the image on the GPU.
4. **Integration**: Combines Iced and wgpu to display the rendered image in the GUI.

---

### **5. Implementation**

#### **Step 1: Basic Iced Setup**
Start by creating a simple Iced application.

```rust
use iced::{Application, Command, Element, Settings, Subscription};
use iced_wgpu::Renderer;

struct ImageViewer {
    // Add state variables here
}

#[derive(Debug, Clone)]
enum Message {
    // Define messages for UI interactions
}

impl Application for ImageViewer {
    type Executor = iced::executor::Default;
    type Message = Message;
    type Theme = iced::Theme;
    type Flags = ();

    fn new(_flags: ()) -> (Self, Command<Self::Message>) {
        (
            ImageViewer {
                // Initialize state variables
            },
            Command::none(),
        )
    }

    fn title(&self) -> String {
        String::from("GPU-Accelerated Image Viewer")
    }

    fn update(&mut self, message: Self::Message) -> Command<Self::Message> {
        match message {
            // Handle messages
        }
        Command::none()
    }

    fn view(&self) -> Element<Self::Message> {
        // Define the UI layout
        iced::widget::container(iced::widget::text("Image Viewer"))
            .center_x()
            .center_y()
            .into()
    }

    fn subscription(&self) -> Subscription<Self::Message> {
        Subscription::none()
    }
}

fn main() -> iced::Result {
    ImageViewer::run(Settings::default())
}
```

This code sets up a minimal Iced application with a centered text label.

---

#### **Step 2: Load an Image**
Use the `image` crate to load an image file into memory.

```rust
use image::GenericImageView;

fn load_image(path: &str) -> Result<image::DynamicImage, String> {
    image::open(path).map_err(|e| e.to_string())
}
```

Call this function in the `new` method of your `ImageViewer` struct to load the image.

---

#### **Step 3: GPU Initialization with wgpu**
Set up wgpu to prepare the GPU for rendering.

1. **Create a wgpu instance**:
   ```rust
   use wgpu::{Instance, InstanceDescriptor, Backends, Adapter, Device, Queue};

   async fn initialize_wgpu() -> (Adapter, Device, Queue) {
       let instance = Instance::new(InstanceDescriptor {
           backends: Backends::PRIMARY,
           ..Default::default()
       });

       let adapter = instance
           .request_adapter(&wgpu::RequestAdapterOptions {
               power_preference: wgpu::PowerPreference::HighPerformance,
               force_fallback_adapter: false,
               compatible_surface: None,
           })
           .await
           .expect("Failed to find an appropriate adapter");

       let (device, queue) = adapter
           .request_device(
               &wgpu::DeviceDescriptor {
                   label: None,
                   features: wgpu::Features::empty(),
                   limits: wgpu::Limits::default(),
               },
               None,
           )
           .await
           .expect("Failed to create device");

       (adapter, device, queue)
   }
   ```

2. **Create a texture from the image**:
   Convert the loaded image into a wgpu texture.

   ```rust
   use wgpu::{Texture, TextureView, TextureDescriptor, TextureFormat, Extent3d};

   fn create_texture(device: &Device, queue: &Queue, image: &image::DynamicImage) -> Texture {
       let rgba = image.to_rgba8();
       let dimensions = image.dimensions();

       let texture_size = Extent3d {
           width: dimensions.0,
           height: dimensions.1,
           depth_or_array_layers: 1,
       };

       let texture = device.create_texture(&TextureDescriptor {
           label: None,
           size: texture_size,
           mip_level_count: 1,
           sample_count: 1,
           dimension: wgpu::TextureDimension::D2,
           format: TextureFormat::Rgba8UnormSrgb,
           usage: wgpu::TextureUsages::TEXTURE_BINDING | wgpu::TextureUsages::COPY_DST,
           view_formats: &[],
       });

       queue.write_texture(
           wgpu::ImageCopyTexture {
               texture: &texture,
               mip_level: 0,
               origin: wgpu::Origin3d::ZERO,
               aspect: wgpu::TextureAspect::All,
           },
           &rgba,
           wgpu::ImageDataLayout {
               offset: 0,
               bytes_per_row: Some(4 * dimensions.0),
               rows_per_image: Some(dimensions.1),
           },
           texture_size,
       );

       texture
   }
   ```

---

#### **Step 4: Integrate wgpu with Iced**
Use the `iced_wgpu` crate to integrate wgpu rendering into Iced.

1. **Create a custom widget**:
   Implement a custom widget to render the texture.

   ```rust
   use iced_wgpu::Renderer;
   use iced::widget::{Canvas, canvas};

   struct ImageWidget {
       texture_view: TextureView,
   }

   impl<Message> canvas::Program<Message> for ImageWidget {
       type State = ();

       fn draw(
           &self,
           _state: &Self::State,
           renderer: &Renderer,
           _theme: &iced::Theme,
           bounds: iced::Rectangle,
           _cursor: iced::mouse::Cursor,
       ) -> Vec<canvas::Geometry> {
           let mut frame = renderer.frame(bounds.size());

           // Draw the texture here using wgpu
           frame.fill_rectangle(
               iced::Point::ORIGIN,
               bounds.size(),
               iced::Color::WHITE,
           );

           vec![frame.into_geometry()]
       }
   }
   ```

2. **Add the custom widget to the UI**:
   Replace the placeholder UI in the `view` method with the custom widget.

---

### **6. Final Steps**
1. **Run the application**:
   Execute the program using `cargo run`.

2. **Test with different images**:
   Modify the image path in the `load_image` function to test with other images.

3. **Optimize and extend**:
   - Add zooming and panning functionality.
   - Support multiple image formats.
   - Improve error handling.

---

### **7. Conclusion**
This guide provides a foundation for building a GPU-accelerated image viewer using Iced and wgpu. While the implementation is simplified, it demonstrates the key concepts of integrating GPU rendering with a GUI framework in Rust. From here, you can expand the application by adding more features, improving performance, and refining the user experience.
