Kako testirati REST API endpoint s pomočjo Postmana?

### 1. **Priprava Okolja**

Najprej potrebujemo Rust okolje in Postman.

- **Namestitev Rust**: Če še nimaš nameščenega Rust, ga lahko namestiš s pomočjo [rustup](https://rustup.rs/).
- **Namestitev Postmana**: Postman lahko namestiš s [uradne strani](https://www.postman.com/downloads/).

### 2. **Ustvarjanje Rust Projekta**

Odpri terminal in ustvari nov Rust projekt:

```bash
cargo new moj_api
cd moj_api
```

### 3. **Dodajanje Odvisnosti**

V `Cargo.toml` dodaj potrebne odvisnosti. Za ustvarjanje REST API bomo uporabili `warp`:

```toml
[dependencies]
warp = "0.3"
tokio = { version = "1", features = ["full"] }
```

### 4. **Ustvarjanje REST API Endpointa**

V `src/main.rs` ustvari preprost REST API endpoint:

```rust
use warp::Filter;

#[tokio::main]
async fn main() {
    // Definiraj pot za GET zahtevo
    let hello = warp::path!("hello" / String)
        .map(|name| format!("Pozdravljen, {}!", name));

    // Zaženi strežnik na portu 3030
    warp::serve(hello)
        .run(([127, 0, 0, 1], 3030))
        .await;
}
```

### 5. **Zagon Strežnika**

Zaženi strežnik s sledečim ukazom:

```bash
cargo run
```

Strežnik bo poslušal na `http://127.0.0.1:3030`.

### 6. **Testiranje s Postmanom**

Odpri Postman in sledi tem korakom:

1. **Ustvari Novo Zahtevo**:
   - Klikni na `New` in izberi `Request`.
   - Poimenuj zahtevo (npr. `Test Hello Endpoint`).

2. **Nastavi Zahtevo**:
   - Izberi `GET` metodo.
   - Vnesi URL: `http://127.0.0.1:3030/hello/Jan`.
   - Klikni na `Send`.

3. **Preveri Odgovor**:
   - V odseku `Body` bi moral videti odgovor: `Pozdravljen, Jan!`.

### 7. **Dodatne Možnosti**

- **Dodajanje Več Endpointov**: Lahko dodajaš več endpointov in jih testiraš na podoben način.
- **Testiranje Različnih Metod**: Preizkusi tudi druge HTTP metode (POST, PUT, DELETE) in preveri, kako se odziva tvoj API.

### 8. **Debugging in Logging**

Za lažje odpravljanje napak lahko dodajaš logge s pomočjo `log` in `env_logger` knjižnic.

```toml
[dependencies]
log = "0.4"
env_logger = "0.9"
```

Inicializiraj logger v `main.rs`:

```rust
use log::info;

#[tokio::main]
async fn main() {
    env_logger::init();

    info!("Strežnik se zaganja...");

    let hello = warp::path!("hello" / String)
        .map(|name| {
            info!("Prejeto ime: {}", name);
            format!("Pozdravljen, {}!", name)
        });

    warp::serve(hello)
        .run(([127, 0, 0, 1], 3030))
        .await;
}
```

