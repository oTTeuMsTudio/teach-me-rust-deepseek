# Build a sea ship construction game
Building a game in Rust where players construct a sea ship in their personal harbor is an excellent way to learn about game development and Rust programming. While designing and implementing this game we'll cover the basics of Rust, object-oriented design, and game logic.

---

### **Step 1: Define the Game Concept**
The game revolves around:
1. A player managing a personal harbor.
2. Building and customizing a sea ship by purchasing or upgrading parts (e.g., hull, sails, cannons).
3. Displaying the ship's current state and allowing interactions.

---

### **Step 2: Set Up the Rust Project**
First, create a new Rust project using Cargo:
```bash
cargo new sea_ship_builder
cd sea_ship_builder
```

Add any necessary dependencies in `Cargo.toml` if needed (e.g., for input handling or logging). For simplicity, we'll use the standard library for now.

---

### **Step 3: Design the Game Structure**
We need to define the core components of the game:
1. **Harbor**: Represents the player's harbor where ships are built.
2. **Ship**: Represents the player's sea ship with customizable parts.
3. **Parts**: Components like hull, sails, cannons, etc., that can be upgraded.

Here’s how we can structure these components:

#### **Harbor Struct**
The harbor will manage the player's resources and ship-building process.
```rust
pub struct Harbor {
    pub gold: u32,
    pub ship: Option<Ship>,
}

impl Harbor {
    pub fn new(gold: u32) -> Self {
        Harbor { gold, ship: None }
    }

    pub fn build_ship(&mut self) {
        if self.ship.is_some() {
            println!("You already have a ship!");
            return;
        }
        self.ship = Some(Ship::new());
        println!("A new ship has been constructed!");
    }

    pub fn upgrade_ship(&mut self) {
        if let Some(ship) = &mut self.ship {
            ship.upgrade();
        } else {
            println!("You need to build a ship first!");
        }
    }

    pub fn display_status(&self) {
        println!("Harbor Status:");
        println!("Gold: {}", self.gold);
        if let Some(ship) = &self.ship {
            println!("Ship: {}", ship);
        } else {
            println!("No ship built yet.");
        }
    }
}
```

#### **Ship Struct**
The ship will have customizable parts and methods for upgrading them.
```rust
pub struct Ship {
    pub hull: String,
    pub sails: String,
    pub cannons: u8,
}

impl Ship {
    pub fn new() -> Self {
        Ship {
            hull: "Wooden Hull".to_string(),
            sails: "Basic Sails".to_string(),
            cannons: 2,
        }
    }

    pub fn upgrade(&mut self) {
        self.hull = "Reinforced Hull".to_string();
        self.sails = "Advanced Sails".to_string();
        self.cannons += 2;
        println!("Your ship has been upgraded!");
    }
}

impl std::fmt::Display for Ship {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(
            f,
            "Hull: {}, Sails: {}, Cannons: {}",
            self.hull, self.sails, self.cannons
        )
    }
}
```

---

### **Step 4: Implement the Game Logic**
Now, let’s implement the main game loop where the player interacts with the harbor.

#### **Main Function**
```rust
use std::io;

fn main() {
    let mut harbor = Harbor::new(100);

    loop {
        println!("\nWhat would you like to do?");
        println!("1. Build a Ship");
        println!("2. Upgrade Ship");
        println!("3. Display Status");
        println!("4. Exit");

        let mut choice = String::new();
        io::stdin().read_line(&mut choice).expect("Failed to read line");
        let choice: u32 = match choice.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("Invalid input. Please enter a number.");
                continue;
            }
        };

        match choice {
            1 => harbor.build_ship(),
            2 => harbor.upgrade_ship(),
            3 => harbor.display_status(),
            4 => {
                println!("Goodbye!");
                break;
            }
            _ => println!("Invalid option. Please try again."),
        }
    }
}
```

---

### **Step 5: Run the Game**
Build and run the game using:
```bash
cargo run
```

You should see a menu where you can build a ship, upgrade it, check its status, or exit the game.

---

### **Step 6: Enhance the Game**
To make the game more engaging, consider adding:
1. **Resource Management**: Add costs for building and upgrading ships.
   - Example: Deduct gold when building/upgrading.
2. **Customizable Parts**: Allow players to choose specific upgrades for hull, sails, and cannons.
3. **Save/Load System**: Use a file or database to save the player's progress.
4. **Graphics**: Integrate a library like `ggez` or `bevy` for a graphical interface.

---

### **Final Notes**
This is a foundational implementation of a ship-building game in Rust. As you progress, you can expand the game mechanics, add complexity, and polish the user experience. Rust's strong type system and performance make it an excellent choice for game development, especially for systems-level tasks like resource management and physics simulations.
