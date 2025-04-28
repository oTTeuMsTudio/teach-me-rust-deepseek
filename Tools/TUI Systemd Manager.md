#TUI Systemd Manager
To create a Rust program for managing systemd services through a TUI (Terminal User Interface), we will leverage the following:

1. **D-Bus API**: Systemd exposes its functionality via D-Bus, so we'll use the `zbus` crate to interact with it.
2. **TUI Framework**: We'll use the `tui` crate to build the terminal user interface.
3. **Event Handling**: The `crossterm` crate will help handle terminal events like key presses.

Below is a step-by-step guide to building this tool:

---

### Step 1: Set Up the Project
Create a new Rust project:
```bash
cargo new systemd-tui-manager
cd systemd-tui-manager
```

Add the necessary dependencies to your `Cargo.toml`:
```toml
[dependencies]
zbus = "3.0" # For interacting with D-Bus
tokio = { version = "1", features = ["full"] } # Async runtime for zbus
tui = "0.18" # Terminal UI library
crossterm = "0.26" # Terminal event handling
anyhow = "1.0" # Error handling
```

---

### Step 2: Define the Application Structure
We'll create a modular structure for clarity:
- **main.rs**: Entry point of the application.
- **dbus.rs**: Handles D-Bus communication with systemd.
- **tui.rs**: Manages the terminal UI and user interactions.

---

### Step 3: Implement D-Bus Communication
In `src/dbus.rs`, implement functions to interact with systemd via D-Bus:
```rust
use zbus::{Connection, Result};
use zvariant::Value;

pub async fn list_services() -> Result<Vec<String>> {
    let connection = Connection::system().await?;
    let proxy = connection.dbus_proxy().await?;
    let services: Vec<String> = proxy
        .list_names()
        .await?
        .into_iter()
        .filter(|name| name.starts_with("org.freedesktop.systemd1"))
        .collect();
    Ok(services)
}

pub async fn start_service(service_name: &str) -> Result<()> {
    let connection = Connection::system().await?;
    let proxy = connection.dbus_proxy().await?;
    proxy.call_method(
        "org.freedesktop.systemd1.Manager",
        "StartUnit",
        &(service_name, "replace"),
    ).await?;
    Ok(())
}

// Add similar functions for stop, restart, enable, disable, etc.
```

---

### Step 4: Build the TUI
In `src/tui.rs`, create the terminal UI:
```rust
use crossterm::{
    event::{self, Event, KeyCode},
    execute,
    terminal::{disable_raw_mode, enable_raw_mode, EnterAlternateScreen, LeaveAlternateScreen},
};
use tui::{
    backend::CrosstermBackend,
    layout::{Constraint, Direction, Layout},
    widgets::{Block, Borders, List, ListItem},
    Terminal,
};
use std::io;

pub struct App {
    pub items: Vec<String>,
    pub selected: usize,
}

impl App {
    pub fn new(items: Vec<String>) -> App {
        App {
            items,
            selected: 0,
        }
    }
}

pub fn run_tui(app: &mut App) -> Result<(), io::Error> {
    enable_raw_mode()?;
    let mut stdout = io::stdout();
    execute!(stdout, EnterAlternateScreen)?;
    let backend = CrosstermBackend::new(stdout);
    let mut terminal = Terminal::new(backend)?;

    loop {
        terminal.draw(|f| {
            let chunks = Layout::default()
                .direction(Direction::Vertical)
                .constraints([Constraint::Percentage(100)].as_ref())
                .split(f.size());

            let items: Vec<ListItem> = app
                .items
                .iter()
                .map(|i| ListItem::new(i.as_str()))
                .collect();

            let list = List::new(items)
                .block(Block::default().borders(Borders::ALL).title("Systemd Services"))
                .highlight_style(tui::style::Style::default().bg(tui::style::Color::Yellow));

            f.render_widget(list, chunks[0]);
            if !app.items.is_empty() {
                f.set_cursor(
                    chunks[0].x + app.selected as u16 + 1,
                    chunks[0].y + app.selected as u16 + 1,
                );
            }
        })?;

        if let Event::Key(key) = event::read()? {
            match key.code {
                KeyCode::Char('q') => break,
                KeyCode::Down => app.selected = (app.selected + 1) % app.items.len(),
                KeyCode::Up => {
                    if app.selected > 0 {
                        app.selected -= 1;
                    }
                }
                _ => {}
            }
        }
    }

    disable_raw_mode()?;
    execute!(terminal.backend_mut(), LeaveAlternateScreen)?;
    Ok(())
}
```

---

### Step 5: Integrate Everything in `main.rs`
Combine the D-Bus and TUI logic in `src/main.rs`:
```rust
mod dbus;
mod tui;

use dbus::list_services;
use tui::{App, run_tui};
use tokio;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // Fetch systemd services
    let services = list_services().await?;
    let mut app = App::new(services);

    // Run the TUI
    if let Err(err) = run_tui(&mut app) {
        eprintln!("Error: {}", err);
    }

    Ok(())
}
```

---

### Step 6: Extend Functionality
You can extend the tool by adding more D-Bus methods for:
- Viewing logs (`journalctl` integration).
- Enabling/disabling services.
- Restarting services.

For example, to add a `restart_service` function:
```rust
pub async fn restart_service(service_name: &str) -> Result<()> {
    let connection = Connection::system().await?;
    let proxy = connection.dbus_proxy().await?;
    proxy.call_method(
        "org.freedesktop.systemd1.Manager",
        "RestartUnit",
        &(service_name, "replace"),
    ).await?;
    Ok(())
}
```

---

### Step 7: Run the Program
Build and run the program:
```bash
cargo run
```

---

### Summary
This program provides a basic TUI for managing systemd services using Rust. You can further enhance it by:
- Adding error handling and feedback in the UI.
- Supporting more systemd operations.
- Improving the UI with additional widgets and layouts.
