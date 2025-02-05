Kako napisati osnoven spletni strežnik z uporabo programja Actix Web. 


### 2. Pisanje Osnovnega Spletnega Strežnika

Odprite `src/main.rs` in napišite naslednjo kodo:

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};

async fn pozdravi() -> impl Responder {
    HttpResponse::Ok().body("Pozdravljen, svet!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(pozdravi))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### 3. Razlaga Kode

- **`use actix_web::{web, App, HttpServer, Responder, HttpResponse};`**: Uvozimo potrebne komponente iz Actix Web.
- **`async fn pozdravi() -> impl Responder { ... }`**: Definiramo asinhrono funkcijo, ki vrne odgovor (v tem primeru "Pozdravljen, svet!").
- **`#[actix_web::main]`**: Ta atribut omogoča asinhrono izvajanje `main` funkcije.
- **`HttpServer::new(|| { ... })`**: Ustvarimo nov spletni strežnik.
- **`App::new().route("/", web::get().to(pozdravi))`**: Definiramo pot (`/`) in metodo (`GET`), ki bo klicala funkcijo `pozdravi`.
- **`.bind("127.0.0.1:8080")?`**: Strežnik povežemo na lokalni naslov in vrata 8080.
- **`.run().await``: Zaženemo strežnik in čakamo na njegovo izvajanje.

### 4. Zagon Strežnika

Zaženite strežnik z ukazom:

```bash
cargo run
```

Odprite brskalnik in pojdite na `http://127.0.0.1:8080`. Videli bi sporočilo "Pozdravljen, svet!".

### 5. Dodajanje Podatkovne Strukture

Če želite dodati podatkovno strukturo (npr. podatkovno bazo), lahko uporabite `web::Data`. Tukaj je primer:

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use std::sync::Mutex;

struct AppState {
    ime: String,
}

async fn pozdravi(data: web::Data<Mutex<AppState>>) -> impl Responder {
    let data = data.lock().unwrap();
    HttpResponse::Ok().body(format!("Pozdravljen, {}!", data.ime))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let podatki = web::Data::new(Mutex::new(AppState {
        ime: "Svet".to_string(),
    }));

    HttpServer::new(move || {
        App::new()
            .app_data(podatki.clone())
            .route("/", web::get().to(pozdravi))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### 6. Razlaga Dodatne Kode

- **`struct AppState { ime: String }`**: Definiramo strukturo za shranjevanje stanja.
- **`web::Data<Mutex<AppState>>`**: Uporabimo `Mutex` za varno deljenje stanja med nitmi.
- **`app_data(podatki.clone())`**: Dodamo stanje v aplikacijo.

### 7. Zagon in Testiranje

Ponovno zaženite strežnik in obiščite `http://127.0.0.1:8080`. Tokrat bi morali videti sporočilo "Pozdravljen, Svet!".

