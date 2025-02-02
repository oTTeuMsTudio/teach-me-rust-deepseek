Tukaj se učimo, kako narediti prvi asinhroni API klic v nizkonivojskem jeziku Rust. Za začetek bomo uporabili knjižnico `reqwest`, ki je ena najbolj priljubljenih knjižnic za HTTP zahteve. Poleg tega bomo uporabili `tokio` kot asinhroni runtime, saj `reqwest` zahteva asinhrono okolje.

### Korak 1: Nastavitev projekta

Najprej ustvari nov Rust projekt:

```bash
cargo new moj_prvi_async_api_klic
cd moj_prvi_async_api_klic
```

### Korak 2: Dodajte odvisnosti

Odpri `Cargo.toml` in dodajte `dependencies`:

```toml
[dependencies]
reqwest = { version = "0.11", features = ["json"] }
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

- `reqwest`: Za HTTP zahteve.
- `tokio`: Asinhroni runtime.
- `serde` in `serde_json`: Za delo s JSON podatki.

### Korak 3: Napišite kodo

Odpri `src/main.rs` in napišite naslednjo kodo:

```rust
use reqwest::Error;
use serde::Deserialize;

// Definirajte strukturo za JSON odgovor
#[derive(Deserialize, Debug)]
struct ApiOdgovor {
    // Primer: Predpostavimo, da API vrne polje "message"
    message: String,
}

#[tokio::main]
async fn main() -> Result<(), Error> {
    // URL API-ja, ki ga bomo klicali
    let url = "https://api.example.com/data"; // Zamenjajte s pravim URL-jem

    // Asinhroni GET poziv
    let odgovor = reqwest::get(url).await?;

    // Preverite, ali je zahteva uspešna
    if odgovor.status().is_success() {
        // Parsiraj JSON odgovor
        let api_odgovor: ApiOdgovor = odgovor.json().await?;
        println!("Sporočilo iz API-ja: {}", api_odgovor.message);
    } else {
        println!("Zahteva ni bila uspešna: {}", odgovor.status());
    }

    Ok(())
}
```

### Korak 4: Poženite projekt

Zaženite projekt z ukazom:

```bash
cargo run
```

### Razlaga kode

1. **Struct `ApiOdgovor`**: To je struktura, ki predstavlja JSON odgovor API-ja. V tem primeru predpostavljamo, da API vrne polje `message`.

2. **`#[tokio::main]`**: Ta atribut omogoča asinhrono izvajanje funkcije `main`.

3. **`reqwest::get(url).await?`**: To je asinhroni GET poziv na podani URL. `await` čaka na zaključek zahteve.

4. **`odgovor.json().await?`**: Parsira JSON odgovor v strukturo `ApiOdgovor`.

5. **`println!`**: Izpiše sporočilo, ki ga vrne API.

### Primer uporabe

Če želite testirati s pravim API-jem, lahko uporabite brezplačen API, kot je npr. `https://api.adviceslip.com/advice`, ki vrne naključen nasvet. V tem primeru bi struktura `ApiOdgovor` izgledala takole:

```rust
#[derive(Deserialize, Debug)]
struct ApiOdgovor {
    slip: Slip,
}

#[derive(Deserialize, Debug)]
struct Slip {
    advice: String,
    id: u32,
}
```

In URL bi bil:

```rust
let url = "https://api.adviceslip.com/advice";
```

### Zaključek

To je osnovni primer asinhronega API klica v Rustu. Rust je nizkonivojski programski jezik, ki omogoča učinkovito delo z asinhronimi operacijami. V nadaljevanju bomo razširili ta primer z bolj zapletenimi funkcionalnostmi, kot so obdelava napak, avtentikacija itd.
