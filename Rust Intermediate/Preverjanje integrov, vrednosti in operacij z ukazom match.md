### Kaj je Pattern Matching?
Z ukazom `match` preverimo ali integer, ali vrednost, ali operacijo. V drugih jezikih je podoben ukaz `switch`, vendar je bolj fleksibilen in varen.

---

### 1. Preverjanje integra
Primer: Preverimo vrednost celega števila ali `integra` in izvedemo različne ukaze glede na njegovo vrednost.

```rust
fn main() {
    let stevilo = 3;

    match stevilo {
        1 => println!("Ena"),
        2 => println!("Dva"),
        3 => println!("Tri"),
        _ => println!("Neznano število"), // `_` je "wildcard" za vse ostale primere
    }
}
```

**Razlaga:**
- `match` preveri `stevilo`.
- Če je `število` enako `1`, izpiše `"Ena"`.
- Če je `število` enako `2`, izpiše `"Dva"`.
- Če je `število` enako `3`, izpiše `"Tri"`.
- `_` pokriva vse ostale možnosti (kot `else` v `if` stavku).

---

### 2. Preverjanje vrednosti
Tip `Option` se uporablja, ko vrednost obstaja (`Some`) ali pa je odsotna (`None`). Primer:

```rust
fn preveri_opcijo(opcija: Option<i32>) {
    match opcija {
        Some(vrednost) => println!("Vrednost je: {}", vrednost),
        None => println!("Ni vrednosti"),
    }
}

fn main() {
    let nekaj = Some(42);
    let nič = None;

    preveri_opcijo(nekaj); // Izpiše: Vrednost je: 42
    preveri_opcijo(nič);   // Izpiše: Ni vrednosti
}
```

**Razlaga:**
- `Option` je lahko `Some(42)` ali `None`.
- `match` preveri, ali je vrednost prisotna (`Some`) ali ne (`None`).

---

### 3. Preverjanje operacij
Tip `Result` se uporablja za obdelavo operacij, ki lahko uspejo (`Ok`) ali spodletijo (`Err`). Primer:

```rust
fn preveri_rezultat(rezultat: Result<i32, String>) {
    match rezultat {
        Ok(vrednost) => println!("Uspeh: {}", vrednost),
        Err(sporočilo) => println!("Napaka: {}", sporočilo),
    }
}

fn main() {
    let uspeh = Ok(42);
    let napaka = Err("Nekaj je šlo narobe".to_string());

    preveri_rezultat(uspeh);  // Izpiše: Uspeh: 42
    preveri_rezultat(napaka); // Izpiše: Napaka: Nekaj je šlo narobe
}
```

**Razlaga:**
- `Result` je lahko `Ok(42)` (uspeh) ali `Err(sporočilo)` (napaka).
- `match` preveri, ali je operacija uspela ali ne.

