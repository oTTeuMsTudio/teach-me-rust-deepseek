Rust ima močan sistem za obvladovanje napak, ki temelji na dveh glavnih konceptih: `Result` in `Option`. V slovenščini bi lahko te koncepte poimenovali "Rezultat" in "Možnost". Poglejmo si, kako delujejo in kako jih uporabljamo.

### 1. `Result` (Rezultat)

`Result` je enum (naštevalni tip), ki se uporablja za obvladovanje operacij, ki lahko uspejo ali spodletijo. Ima dve možnosti:

- `Ok(T)` - operacija je uspela in vsebuje rezultat.
- `Err(E)` - operacija je spodletela in vsebuje napako.

#### Primer uporabe `Result`:

```rust
fn deljenje(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Deljenje z nič ni dovoljeno.".to_string())
    } else {
        Ok(a / b)
    }
}

fn main() {
    match deljenje(10.0, 0.0) {
        Ok(rezultat) => println!("Rezultat: {}", rezultat),
        Err(napaka) => println!("Napaka: {}", napaka),
    }
}
```

V tem primeru funkcija `deljenje` vrne `Ok` z rezultatom, če deljenje uspe, ali `Err` z opisom napake, če deljenje z nič ni mogoče.

### 2. `Option` (Možnost)

`Option` je podoben `Result`, vendar se uporablja, ko je vrednost lahko prisotna (`Some`) ali odsotna (`None`).

#### Primer uporabe `Option`:

```rust
fn poisci_indeks(vrednost: i32, seznam: Vec<i32>) -> Option<usize> {
    for (indeks, &v) in seznam.iter().enumerate() {
        if v == vrednost {
            return Some(indeks);
        }
    }
    None
}

fn main() {
    let seznam = vec![1, 2, 3, 4, 5];
    match poisci_indeks(3, seznam) {
        Some(indeks) => println!("Vrednost najdena na indeksu: {}", indeks),
        None => println!("Vrednost ni bila najdena."),
    }
}
```

Tu funkcija `poisci_indeks` vrne `Some` z indeksom, če je vrednost najdena, ali `None`, če vrednosti ni v seznamu.

### 3. Uporaba `unwrap` in `expect`

Včasih želimo vrednost iz `Result` ali `Option` takoj pridobiti, ne da bi preverjali, ali je operacija uspela. V tem primeru lahko uporabimo `unwrap` ali `expect`.

- `unwrap()` - vrne vrednost, če je `Ok` ali `Some`, sicer panično zaključi program.
- `expect("sporočilo")` - podobno kot `unwrap`, vendar omogoča dodajanje sporočila, ki se izpiše ob napaki.

#### Primer uporabe `unwrap` in `expect`:

```rust
fn main() {
    let rezultat = deljenje(10.0, 2.0).unwrap();
    println!("Rezultat: {}", rezultat);

    let indeks = poisci_indeks(3, vec![1, 2, 3, 4, 5]).expect("Vrednost ni bila najdena.");
    println!("Indeks: {}", indeks);
}
```

Uporaba `unwrap` in `expect` je priročna, vendar moramo biti previdni, saj lahko povzročijo paniko, če pride do napake.

### 4. Propagacija napak z `?`

Rust omogoča propagacijo napak z uporabo operatorja `?`. To pomeni, da če pride do napake, se napaka vrne iz funkcije, brez potrebe po eksplicitnem pisanju `match` stavkov.

#### Primer uporabe `?`:

```rust
fn deljenje(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Deljenje z nič ni dovoljeno.".to_string())
    } else {
        Ok(a / b)
    }
}

fn izracunaj() -> Result<f64, String> {
    let rezultat = deljenje(10.0, 0.0)?;
    Ok(rezultat)
}

fn main() {
    match izracunaj() {
        Ok(rezultat) => println!("Rezultat: {}", rezultat),
        Err(napaka) => println!("Napaka: {}", napaka),
    }
}
```

V tem primeru, če `deljenje` vrne `Err`, se napaka propagira iz funkcije `izracunaj`.

### Zaključek

Obvladovanje napak v Rustu je močno in fleksibilno. Z uporabo `Result` in `Option` lahko učinkovito obvladujemo napake in zagotavljamo, da je naša koda robustna in zanesljiva. Uporaba `unwrap` in `expect` je priročna, vendar je bolje uporabljati `match` ali `?` za bolj kontrolirano obvladovanje napak.

