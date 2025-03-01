# Bevy ECS
Bevy is a modern, open-source game engine written in Rust. One of its core features is its **Entity-Component-System (ECS)** architecture, which is designed to handle large amounts of data and logic efficiently. Understanding Bevy's ECS is essential for building games or simulations with the engine.

Let’s break down the key concepts of Bevy's ECS: **Entities**, **Components**, and **Systems**.

---

### 1. **Entities**
An **entity** represents an object in your game world. Think of it as a unique identifier for something in your game, like a player character, enemy, or even a UI element. By itself, an entity doesn’t have any properties or behavior; it’s just a label.

In Bevy, entities are lightweight and can be created using the `Commands` resource during system execution.

```rust
commands.spawn(); // Creates a new entity.
```

You can also attach components to entities, which define their properties and behaviors.

---

### 2. **Components**
A **component** is a piece of data attached to an entity. Components allow you to define what an entity *is* or *has*. For example:
- A `Position` component might store the coordinates of an entity.
- A `Health` component might store the current health of a character.

Components are plain old Rust structs or types that implement the `Component` trait. Here’s an example:

```rust
#[derive(Component)]
struct Position {
    x: f32,
    y: f32,
}

#[derive(Component)]
struct Health {
    value: u32,
}
```

To add components to an entity, you use the `Commands` API:

```rust
commands.spawn()
    .insert(Position { x: 0.0, y: 0.0 })
    .insert(Health { value: 100 });
```

Now this entity has both a `Position` and a `Health` component.

---

### 3. **Systems**
A **system** is a function that operates on entities and their components. Systems are where the logic of your game lives. They run in parallel whenever possible, making Bevy's ECS highly efficient.

#### System Basics
Systems are defined as functions that take parameters, which are automatically provided by Bevy's ECS. These parameters include:
- Queries to access entities and their components.
- Resources to manage global state.
- Events to handle communication between systems.

Here’s an example of a simple system that updates the position of all entities with a `Velocity` component:

```rust
fn move_system(
    mut query: Query<(&mut Position, &Velocity)>, // Query for entities with Position and Velocity components
) {
    for (mut position, velocity) in query.iter_mut() {
        position.x += velocity.x;
        position.y += velocity.y;
    }
}
```

#### Adding Systems to the App
To register a system, you add it to the app using the `add_systems` method:

```rust
App::new()
    .add_systems(move_system)
    .run();
```

---

### 4. **Queries**
Queries are how you access and filter entities based on their components. In the example above, the `Query<(&mut Position, &Velocity)>` retrieves all entities that have both a `Position` and a `Velocity` component.

You can also use filters to narrow down your queries. For example, to only update entities with a `Health` component greater than zero:

```rust
fn heal_system(
    mut query: Query<&mut Health, With<Health>>, // Only entities with a Health component
) {
    for mut health in query.iter_mut() {
        if health.value > 0 {
            health.value += 1; // Heal the entity
        }
    }
}
```

---

### 5. **Resources**
While components belong to specific entities, **resources** are global and shared across all systems. They are used for things like game settings, timers, or anything that isn’t tied to a single entity.

To create a resource, define a struct and add it to the app:

```rust
#[derive(Resource)]
struct GameTime {
    seconds: f32,
}

App::new()
    .insert_resource(GameTime { seconds: 0.0 }) // Add the resource to the app
    .run();
```

You can then access resources in systems:

```rust
fn time_system(mut game_time: ResMut<GameTime>) {
    game_time.seconds += 1.0; // Increment the game time
}
```

---

### 6. **Events**
Events allow systems to communicate without directly referencing each other. This promotes loose coupling and better modularity.

To use events:
1. Define an event type:
   ```rust
   #[derive(Event)]
   struct DamageEvent {
       amount: u32,
   }
   ```

2. Send an event from one system:
   ```rust
   fn attack_system(mut events: EventWriter<DamageEvent>) {
       events.send(DamageEvent { amount: 10 }); // Send a damage event
   }
   ```

3. Receive the event in another system:
   ```rust
   fn handle_damage_system(
       mut events: EventReader<DamageEvent>,
       mut query: Query<&mut Health>,
   ) {
       for event in events.read() {
           for mut health in query.iter_mut() {
               health.value -= event.amount; // Apply damage
           }
       }
   }
   ```

---

### Putting It All Together
Here’s a complete example of a simple game loop using Bevy's ECS:

```rust
use bevy::prelude::*;

// Components
#[derive(Component)]
struct Position {
    x: f32,
    y: f32,
}

#[derive(Component)]
struct Velocity {
    x: f32,
    y: f32,
}

// Systems
fn setup(mut commands: Commands) {
    commands.spawn()
        .insert(Position { x: 0.0, y: 0.0 })
        .insert(Velocity { x: 1.0, y: 2.0 });
}

fn move_system(mut query: Query<(&mut Position, &Velocity)>) {
    for (mut position, velocity) in query.iter_mut() {
        position.x += velocity.x;
        position.y += velocity.y;
    }
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems((setup, move_system))
        .run();
}
```

---

### Key Takeaways
- **Entities** are unique identifiers for objects in your game.
- **Components** define the data and properties of entities.
- **Systems** contain the logic that operates on entities and components.
- **Queries** allow you to access and filter entities based on their components.
- **Resources** are global data shared across systems.
- **Events** enable communication between systems.

By mastering these concepts, you’ll be well-equipped to build complex and efficient games with Bevy!
