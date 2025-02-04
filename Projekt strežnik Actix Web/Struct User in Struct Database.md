Kako uporabljati `#[derive(Serialize, Deserialize, Debug, Clone)]` za strukturo `User` in kako bi lahko izgledala struktura `Database`?
### 1. Struktura `User` z `#[derive]`

`#[derive]` atribut uporabimo, da samodejno implementiramo različne lastnosti (`traits`) za `struct-e`. V našem primeru želimo, da ima struct `User` implementirane lastnosti `Serialize`, `Deserialize`, `Debug` in `Clone`.

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug, Clone)]
struct User {
    id: u32,
    username: String,
    email: String,
}
```

#### Razlaga:
- **`Serialize` in `Deserialize`**: Ti lastnosti omogočajo, da strukturo `User` pretvorimo v JSON ali druge formate (serializacija) in obratno (deserializacija). Za to uporabljamo knjižnico `serde`.
- **`Debug`**: Ta lastnost omogoča, da strukturo izpišemo z uporabo `{:?}` ali `{:#?}`, kar je uporabno za odpravljanje napak.
- **`Clone`**: Ta lastnost omogoča, da ustvarimo kopijo strukture z metodo `.clone()`.

### 2. Struktura `Database`

Struktura `Database` bi lahko predstavljala zbirko uporabnikov (`User`). Na primer, lahko bi bila to preprosta struktura, ki vsebuje vektor uporabnikov.

```rust
struct Database {
    users: Vec<User>,
}
```

#### Razlaga:
- **`users: Vec<User>`**: To je vektor (dinamično polje) uporabnikov. Vsak element vektorja je struktura `User`.

### 3. Primer uporabe

Tukaj je primer, kako bi lahko uporabil te strukture:

```rust
fn main() {
    // Ustvarimo nekaj uporabnikov
    let user1 = User {
        id: 1,
        username: String::from("janez"),
        email: String::from("janez@example.com"),
    };

    let user2 = User {
        id: 2,
        username: String::from("meta"),
        email: String::from("meta@example.com"),
    };

    // Ustvarimo bazo podatkov z uporabniki
    let mut database = Database {
        users: vec![user1.clone(), user2.clone()],
    };

    // Dodamo še enega uporabnika
    let user3 = User {
        id: 3,
        username: String::from("alen"),
        email: String::from("alen@example.com"),
    };
    database.users.push(user3);

    // Izpišemo bazo podatkov
    println!("{:#?}", database);

    // Serializacija uporabnika v JSON
    let json_string = serde_json::to_string(&user1).unwrap();
    println!("Serialized user: {}", json_string);

    // Deserializacija uporabnika iz JSON
    let deserialized_user: User = serde_json::from_str(&json_string).unwrap();
    println!("Deserialized user: {:?}", deserialized_user);
}
```

#### Razlaga:
- **`user1.clone()`**: Uporabimo metodo `clone()`, da ustvarimo kopijo uporabnika, ker želimo uporabnika dodati v vektor, ne pa ga premakniti.
- **`serde_json::to_string(&user1)`**: Serializiramo uporabnika v JSON niz.
- **`serde_json::from_str(&json_string)`**: Deserializiramo JSON niz nazaj v strukturo `User`.

### 4. Namestitev potrebnih knjižnic

Za uporabo `serde` in `serde_json` moraš dodati te knjižnice v svoj `Cargo.toml`:

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

