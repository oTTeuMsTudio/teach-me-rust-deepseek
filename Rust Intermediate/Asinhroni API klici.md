Rust ima odlično podporo za asinhrono programiranje, predvsem s pomočjo knjižnice `tokio` ali `async-std`. V tem primeru bomo uporabili `tokio`, ker je ta knjižnica najbolj priljubljena.

### 1. Namestitev odvisnosti

Najprej moramo v `Cargo.toml` dodati potrebne odvisnosti:

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
reqwest = "0.11"
```

- `tokio`: Asinhroni runtime za Rust.
- `reqwest`: Knjižnica za HTTP kliente, ki podpira asinhrone klice.

### 2. Osnovni asinhroni API klic

Nato napišemo osnovni primer asinhronega API klica. Uporabili bomo `reqwest` za pošiljanje HTTP zahtev in `tokio` za upravljanje asinhronega izvajanja.

```rust
use reqwest::Error;

#[tokio::main]
async fn main() -> Result<(), Error> {
    // URL API-ja, ki ga želimo klicati
    let url = "https://jsonplaceholder.typicode.com/posts/1";

    // Asinhrono pošljemo GET zahtevo
    let response = reqwest::get(url).await?;

    // Preverimo, ali je odgovor uspešen
    if response.status().is_success() {
        // Preberemo telo odgovora kot niz
        let body = response.text().await?;
        println!("Odgovor: {}", body);
    } else {
        println!("Zahteva ni uspela s statusom: {}", response.status());
    }

    Ok(())
}
```

### 3. Razlaga kode

- `#[tokio::main]`: Ta atribut omogoča, da je `main` funkcija asinhrona. `tokio` runtime bo upravljal asinhrono izvajanje.
- `reqwest::get(url).await?`: Pošljemo asinhrono GET zahtevo na podani URL. `.await` čaka na dokončanje zahteve.
- `response.text().await?`: Preberemo telo odgovora kot niz. Spet uporabimo `.await`, ker je ta operacija asinhrona.
- `response.status()`: Preverimo statusni kode odgovora, da ugotovimo, ali je bil klic uspešen.

### 4. Zagon programa

Ko napišete kodo, jo lahko zaženete z ukazom:

```bash
cargo run
```

Če je vse pravilno nastavljeno, bi moral program izpisati vsebino API odgovora v terminal.

### 5. Dodatni primeri

Če želite poslati kompleksnejše zahteve (npr. POST z JSON vsebino), lahko uporabite naslednji primer:

```rust
use reqwest::Error;
use serde_json::json;

#[tokio::main]
async fn main() -> Result<(), Error> {
    let client = reqwest::Client::new();
    let url = "https://jsonplaceholder.typicode.com/posts";

    // Ustvarimo JSON telo za POST zahtevo
    let new_post = json!({
        "title": "Moja nova objava",
        "body": "To je vsebina moje objave.",
        "userId": 1,
    });

    // Asinhrono pošljemo POST zahtevo
    let response = client
        .post(url)
        .json(&new_post)
        .send()
        .await?;

    if response.status().is_success() {
        let body = response.text().await?;
        println!("Odgovor: {}", body);
    } else {
        println!("
