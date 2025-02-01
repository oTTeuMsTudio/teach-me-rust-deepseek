Danes ponavljamo `Result` in `Option`.

### 1. `Option`

`Option` je enum (izčrpen seznam), ki se uporablja, ko lahko vrednost obstaja ali pa ne. To je zelo uporabno, ko želimo izogniti napakam, povezanim z `null` vrednostmi, ki so pogoste v drugih jezikih.

```rust
enum Option<T> {
    Some(T),  // Vrednost obstaja
    None,     // Vrednost ne obstaja
}
```

#### Primer uporabe `Option`:

```rust
fn najdi_stevilo(seznam: Vec<i32>, iskano: i32) -> Option<usize> {
    for (indeks, &vrednost) in seznam.iter().enumerate() {
        if vrednost == iskano {
            return Some(indeks);  // Vrnemo indeks, če najdemo število
        }
    }
    None  // Če števila ni v seznamu, vrnemo None
}

fn main() {
    let seznam = vec![1, 2, 3, 4, 5];
    match najdi_stevilo(seznam, 3) {
        Some(indeks) => println!("Število najdeno na indeksu: {}", indeks),
        None => println!("Število ni bilo najdeno."),
    }
}
```

### 2. `Result`

`Result` je še en enum, ki se uporablja za obravnavanje operacij, ki lahko uspejo ali pa spodletijo. Uporablja se predvsem za obravnavanje napak.

```rust
enum Result<T, E> {
    Ok(T),  // Operacija je uspela, vrnjena je vrednost tipa T
    Err(E), // Operacija je spodletela, vrnjena je napaka tipa E
}
```

#### Primer uporabe `Result`:

```rust
fn deli(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        return Err("Deljenje z nič ni dovoljeno.".to_string());  // Vrnemo napako
    }
    Ok(a / b)  // Vrnemo rezultat deljenja
}

fn main() {
    match deli(10.0, 0.0) {
        Ok(rezultat) => println!("Rezultat deljenja: {}", rezultat),
        Err(napaka) => println!("Napaka: {}", napaka),
    }
}
```

### Povzetek

- **`Option`** se uporablja, ko vrednost lahko obstaja ali pa ne. 
  - `Some(T)` pomeni, da vrednost obstaja.
  - `None` pomeni, da vrednost ne obstaja.

- **`Result`** se uporablja za obravnavanje operacij, ki lahko uspejo ali spodletijo.
  - `Ok(T)` pomeni, da je operacija uspela in vsebuje rezultat.
  - `Err(E)` pomeni, da je operacija spodletela in vsebuje napako.

Oba koncepta sta orodja za obravnavanje negotovosti in napak na varen način, brez uporabe `null` vrednosti.
