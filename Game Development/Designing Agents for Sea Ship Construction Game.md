# Designing Agents for Building Sea Ship Construction Game
To build a sea ship construction game in Rust, we can approach the problem by designing agents (or entities) that interact with each other to simulate the game mechanics. These agents will include players, ships, and the game environment. Lets go through the process of writing this game step by step.

---

### **Step 1: Define the Problem**
The goal is to create a game where:
1. Players construct ships.
2. Ships have properties like size, health, and position.
3. The game environment manages the grid or ocean where ships are placed.
4. Players can interact with the game to place ships, attack opponents, and win.

We'll use Rust's object-oriented features (structs, enums, and traits) to model these components.

---

### **Step 2: Plan the Architecture**
We'll break the game into the following components:
1. **Player**: Represents a player who constructs and manages ships.
2. **Ship**: Represents a ship with attributes like size, health, and position.
3. **GameGrid**: Represents the ocean grid where ships are placed.
4. **Game**: Manages the overall game state and interactions between players.

---

### **Step 3: Implement the Components**

#### **1. Define the Ship Struct**
A ship has properties like size, health, and position. We'll also define methods for placing and attacking ships.

```rust
#[derive(Debug)]
struct Ship {
    name: String,
    size: usize,
    health: usize,
    position: Option<(usize, usize)>, // (row, column)
}

impl Ship {
    fn new(name: &str, size: usize) -> Self {
        Ship {
            name: name.to_string(),
            size,
            health: size, // Health equals the size initially
            position: None,
        }
    }

    fn place(&mut self, row: usize, col: usize) {
        self.position = Some((row, col));
        println!("Placed {} at ({}, {})", self.name, row, col);
    }

    fn is_sunk(&self) -> bool {
        self.health == 0
    }

    fn hit(&mut self) {
        if !self.is_sunk() {
            self.health -= 1;
            println!("{} was hit! Remaining health: {}", self.name, self.health);
        } else {
            println!("{} is already sunk!", self.name);
        }
    }
}
```

---

#### **2. Define the GameGrid Struct**
The game grid represents the ocean where ships are placed. It tracks the positions of all ships.

```rust
struct GameGrid {
    size: usize,
    ships: Vec<Ship>,
}

impl GameGrid {
    fn new(size: usize) -> Self {
        GameGrid {
            size,
            ships: Vec::new(),
        }
    }

    fn add_ship(&mut self, ship: Ship) {
        if let Some((row, col)) = ship.position {
            if row < self.size && col < self.size {
                self.ships.push(ship);
                println!("Added ship to the grid.");
            } else {
                println!("Invalid position for the ship!");
            }
        } else {
            println!("Ship must be placed before adding to the grid!");
        }
    }

    fn attack(&mut self, row: usize, col: usize) -> bool {
        if row >= self.size || col >= self.size {
            println!("Attack out of bounds!");
            return false;
        }

        for ship in &mut self.ships {
            if let Some((ship_row, ship_col)) = ship.position {
                if ship_row == row && ship_col == col {
                    ship.hit();
                    return true;
                }
            }
        }

        println!("Missed!");
        false
    }

    fn display(&self) {
        println!("Game Grid (Size: {}x{})", self.size, self.size);
        for row in 0..self.size {
            for col in 0..self.size {
                let mut cell = ".";
                for ship in &self.ships {
                    if let Some((ship_row, ship_col)) = ship.position {
                        if ship_row == row && ship_col == col {
                            cell = "S";
                            break;
                        }
                    }
                }
                print!("{} ", cell);
            }
            println!();
        }
    }
}
```

---

#### **3. Define the Player Struct**
A player constructs ships and interacts with the game grid.

```rust
struct Player {
    name: String,
    grid: GameGrid,
}

impl Player {
    fn new(name: &str, grid_size: usize) -> Self {
        Player {
            name: name.to_string(),
            grid: GameGrid::new(grid_size),
        }
    }

    fn construct_ship(&mut self, name: &str, size: usize, row: usize, col: usize) {
        let mut ship = Ship::new(name, size);
        ship.place(row, col);
        self.grid.add_ship(ship);
    }

    fn attack(&mut self, opponent: &mut Player, row: usize, col: usize) {
        println!("{} attacks at ({}, {})", self.name, row, col);
        opponent.grid.attack(row, col);
    }

    fn display_grid(&self) {
        println!("Player {}'s Grid:", self.name);
        self.grid.display();
    }
}
```

---

#### **4. Define the Game Struct**
The game manages two players and their interactions.

```rust
struct Game {
    player1: Player,
    player2: Player,
}

impl Game {
    fn new(player1_name: &str, player2_name: &str, grid_size: usize) -> Self {
        Game {
            player1: Player::new(player1_name, grid_size),
            player2: Player::new(player2_name, grid_size),
        }
    }

    fn play(&mut self) {
        println!("Welcome to the Sea Ship Construction Game!");

        // Example: Construct ships for both players
        self.player1.construct_ship("Battleship", 4, 0, 0);
        self.player2.construct_ship("Cruiser", 3, 2, 2);

        // Display initial grids
        self.player1.display_grid();
        self.player2.display_grid();

        // Example: Attack sequence
        self.player1.attack(&mut self.player2, 2, 2); // Hit!
        self.player1.attack(&mut self.player2, 1, 1); // Miss!

        // Display updated grids
        self.player1.display_grid();
        self.player2.display_grid();
    }
}
```

---

### **Step 4: Run the Game**
Finally, we can run the game in the `main` function.

```rust
fn main() {
    let mut game = Game::new("Alice", "Bob", 5);
    game.play();
}
```

---

### **Explanation of Key Concepts**
1. **Encapsulation**: Each component (e.g., `Ship`, `GameGrid`, `Player`) encapsulates its own data and behavior.
2. **Modularity**: The game is divided into reusable and testable components.
3. **Interaction**: Players interact with the game grid to place ships and attack opponents.

This structure provides a foundation for expanding the game with additional features, such as more complex ship types, AI opponents, or multiplayer functionality.
