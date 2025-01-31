Moduli v Rustu so način za organizacijo kode v ločene, logične enote, kar olajša vzdrževanje in ponovno uporabo kode.

### 1. Osnove Modulov

V Rustu lahko ustvarite modul z uporabo ključne besede `mod`. Modul lahko vsebuje funkcije, strukture, nabore, druge module in druge elemente.

```rust
// Definirajmo modul `matematika`
mod matematika {
    // Funkcija znotraj modula
    pub fn seštej(a: i32, b: i32) -> i32 {
        a + b
    }

    // Privatna funkcija (ni dostopna zunaj modula)
    fn privatna_funkcija() {
        println!("To je privatna funkcija.");
    }
}

fn main() {
    // Klic funkcije iz modula
    let vsota = matematika::seštej(5, 3);
    println!("Vsota je: {}", vsota);
}
```

- `mod matematika { ... }`: Definira modul z imenom `matematika`.
- `pub fn seštej(...)`: Funkcija `seštej` je javno dostopna (`pub` pomeni "public").
- `matematika::seštej(...)`: Klic funkcije `seštej` iz modula `matematika`.

### 2. Hierarhija Modulov

Moduli lahko tvorijo hierarhijo, kar pomeni, da lahko modul vsebuje druge module.

```rust
mod matematika {
    pub mod osnovne_operacije {
        pub fn seštej(a: i32, b: i32) -> i32 {
            a + b
        }

        pub fn odštej(a: i32, b: i32) -> i32 {
            a - b
        }
    }
}

fn main() {
    let vsota = matematika::osnovne_operacije::seštej(10, 5);
    let razlika = matematika::osnovne_operacije::odštej(10, 5);
    println!("Vsota: {}, Razlika: {}", vsota, razlika);
}
```

- `matematika::osnovne_operacije::seštej(...)`: Klic funkcije `seštej` iz modula `osnovne_operacije`, ki je del modula `matematika`.

### 3. Uporaba `use` za Poenostavitev

Ključna beseda `use` omogoča, da skrajšamo pot do funkcij ali drugih elementov v modulih.

```rust
mod matematika {
    pub mod osnovne_operacije {
        pub fn seštej(a: i32, b: i32) -> i32 {
            a + b
        }

        pub fn odštej(a: i32, b: i32) -> i32 {
            a - b
        }
    }
}

// Uporaba `use` za poenostavitev
use matematika::osnovne_operacije;

fn main() {
    let vsota = osnovne_operacije::seštej(10, 5);
    let razlika = osnovne_operacije::odštej(10, 5);
    println!("Vsota: {}, Razlika: {}", vsota, razlika);
}
```

- `use matematika::osnovne_operacije;`: Poenostavi dostop do modula `osnovne_operacije`.

### 4. Moduli v Ločenih Datotekah

Ko vaša koda postane večja, lahko module razdelite v ločene datoteke.

```rust
// src/matematika.rs
pub mod osnovne_operacije {
    pub fn seštej(a: i32, b: i32) -> i32 {
        a + b
    }

    pub fn odštej(a: i32, b: i32) -> i32 {
        a - b
    }
}

// src/main.rs
mod matematika;

fn main() {
    let vsota = matematika::osnovne_operacije::seštej(10, 5);
    let razlika = matematika::osnovne_operacije::odštej(10, 5);
    println!("Vsota: {}, Razlika: {}", vsota, razlika);
}
```

- `mod matematika;`: Rust samodejno poišče datoteko `matematika.rs` ali `matematika/mod.rs` in jo vključi kot modul.

### 5. Privatnost v Modulih

Privzeto so vsi elementi v modulu privatni. Če želite, da so dostopni zunaj modula, jih označite z `pub`.

```rust
mod matematika {
    pub fn javna_funkcija() {
        println!("To je javna funkcija.");
    }

    fn privatna_funkcija() {
        println!("To je privatna funkcija.");
    }
}

fn main() {
    matematika::javna_funkcija(); // Deluje
    // matematika::privatna_funkcija(); // Ne bi delovalo, ker je privatna
}
```

### 6. Zunanji Moduli

Če želite uporabljati module iz drugih krat (crates), jih morate najprej dodati v `Cargo.toml` in nato uvoziti z `use`.

```toml
# Cargo.toml
[dependencies]
nekaj_krate = "1.0"
```

```rust
// src/main.rs
use nekaj_krate::nekaj_modula;

fn main() {
    nekaj_modula::nekaj_funkcije();
}
```

### Zaključek

Moduli v Rustu so močno orodje za organizacijo kode. Omogočajo vam, da ločite kodo na manjše, bolj obvladljive dele, kar olajša vzdrževanje in razumevanje kode. Upamo, da vam je ta uvod v module v Rustu v slovenščini koristil! Če imate kakršnakoli vprašanja, vprašajte. 😊
