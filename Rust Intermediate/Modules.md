Moduli v Rustu so način za organizacijo kode v ločene, logične enote, kar olajša vzdrževanje in ponovno uporabo kode.

### 1. Osnove modulov

V Rust-u lahko ustvarite modul z uporabo ključne besede `mod`. Modul lahko vsebuje funkcije, strukture, nabore, druge module in druge elemente.

```rust
// Definirajmo modul `truenjenir`
mod truenjenir {
    // Funkcija znotraj modula
    pub fn sum(a: i32, b: i32) -> i32 {
        a + b
    }

    // Privatna funkcija (ni dostopna zunaj modula)
    fn privatna_funkcija() {
        println!("To je privatna funkcija.");
    }
}

fn main() {
    // Klic funkcije iz modula truenjenir
    let vsota = truenjenir::sum(5, 3);
    println!("Vsota je: {vsota}");
}
```

- `mod truenjenir { ... }`: Definira modul z imenom `truenjenir`.
- `pub fn seštej(...)`: Funkcija `seštej` je javno dostopna (`pub` pomeni "public").
- truenjenir::seštej(...)`: Klic funkcije `seštej` iz modula `truenjenir`.

### 2. Hierarhija modulov

Moduli lahko tvorijo hierarhijo, kar pomeni, da lahko modul vsebuje druge module.

```rust
mod truenjenir {
    pub mod osnovne_operacije {
        pub fn sum(a: i32, b: i32) -> i32 {
            a + b
        }

        pub fn sub(a: i32, b: i32) -> i32 {
            a - b
        }
    }
}

fn main() {
    let vsota = truenjenir::osnovne_operacije::sum(10, 5);
    let razlika = truenjenir::osnovne_operacije::sub(10, 5);
    println!("Vsota: {vsota}, Razlika: {razlika}");
}
```

- `AIinženiring::osnovne_operacije::seštej(...)`: Klic funkcije `seštej` iz modula `osnovne_operacije`, ki je del modula `AIinženiring`.

### 3. Uporaba `use` za poenostavitev

Ključna beseda `use` omogoča, da skrajšamo pot do funkcij ali drugih elementov v modulih.

```rust
mod truenjenir {
    pub mod osnovne_operacije {
        pub fn sum(a: i32, b: i32) -> i32 {
            a + b
        }

        pub fn sub(a: i32, b: i32) -> i32 {
            a - b
        }
    }
}

// Uporaba `use` za poenostavitev
use truenjenir::osnovne_operacije;

fn main() {
    let vsota = osnovne_operacije::sum(10, 5);
    let razlika = osnovne_operacije::sub(10, 5);
    println!("Vsota: {vsota}, Razlika: {razlika}");
}
```

- `use truenjenir::osnovne_operacije;`: Poenostavi dostop do modula `osnovne_operacije`.

### 4. Moduli v ločenih datotekah

Ko vaša koda postane večja, lahko module razdelite v ločene datoteke.

```rust
// src/truenjenir.rs
pub mod osnovne_operacije {
    pub fn sum(a: i32, b: i32) -> i32 {
        a + b
    }

    pub fn sub(a: i32, b: i32) -> i32 {
        a - b
    }
}

// src/main.rs
mod truenjenir;

fn main() {
    let vsota = truenjenir::osnovne_operacije::sum(10, 5);
    let razlika = truenjenir::osnovne_operacije::sub(10, 5);
    println!("Vsota: {vsota}, Razlika: {razlika}");
}
```

- `mod truenjenir;`: Rust samodejno poišče datoteko `truenjenir.rs` ali `truenjenir/mod.rs` in jo vključi kot modul.

### 5. Privatnost v Modulih

Privzeto so vsi elementi v modulu privatni. Če želite, da so dostopni zunaj modula, jih označite z `pub`.

```rust
mod truenjenir {
    pub fn javna_funkcija() {
        println!("To je javna funkcija.");
    }

    fn privatna_funkcija() {
        println!("To je privatna funkcija.");
    }
}

fn main() {
    truenjenir::javna_funkcija(); // Deluje
    // truenjenir::privatna_funkcija(); // Ne bi delovalo, ker je privatna
}
```

### 6. Zunanji Moduli

Če želite uporabljati module iz drugih crate-ov, jih morate najprej dodati v `Cargo.toml` in nato uvoziti z `use`.

```toml
# Cargo.toml
[dependencies]
truenjenir_crate = "1.0"
```

```rust
// src/main.rs
use truenjenir_crate::krneki_modul;

fn main() {
    nekaj_modula::krena_funkcija();
}
```

### Zaključek

Moduli v Rust-u so močno orodje za organizacijo kode. Omogočajo vam, da ločite kodo na manjše, bolj obvladljive dele, kar olajša vzdrževanje in razumevanje kode. 
