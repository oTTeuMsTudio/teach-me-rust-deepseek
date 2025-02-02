### Kaj je Enumeracija?

**Enumeracija** kot tipiziranje z več možnimi variantami. Na primer:

```rust
enum Sporocilo {
    Besedilo(String),
    Stevilo(i32),
    Kombinacija(String, i32),
}
```

V tem primeru imamo enumeracijo `Sporocilo`, ki ima tri variante:
- `Besedilo`, ki vsebuje `String`.
- `Stevilo`, ki vsebuje `i32`.
- `Kombinacija`, ki vsebuje `String` in `i32`.

### Tipiziranje enumeracije
**Pattern Matching** je močna lastnost Rust-a, ki omogoča preverjanje, katera varianta enumeracije je trenutno uporabljena, in hkrati dostop do vrednosti, ki so shranjene v tej varianti.

#### Primer:

```rust
fn obdelaj_sporocilo(sporocilo: Sporocilo) {
    match sporocilo {
        Sporocilo::Besedilo(tekst) => println!("Prejel sem besedilo: {}", tekst),
        Sporocilo::Stevilo(stevilo) => println!("Prejel sem število: {}", stevilo),
        Sporocilo::Kombinacija(tekst, stevilo) => {
            println!("Prejel sem kombinacijo: {} in {}", tekst, stevilo)
        }
    }
}
```

V tem primeru:
- `match` preveri, katera varianta `Sporocilo` je bila poslana.
- Če je to `Besedilo`, izpiše besedilo.
- Če je to `Stevilo`, izpiše število.
- Če je to `Kombinacija`, izpiše oboje.

### Uporaba z Mešanimi Podatki

Enumeracije v Rustu so zelo uporabne, ko imamo opravka z mešanimi podatki. Na primer, če želimo obdelati različne tipe sporočil, lahko uporabimo `match`, da obravnavamo vsako varianto posebej.

#### Dodaten Primer:

```rust
fn main() {
    let sporocilo1 = Sporocilo::Besedilo(String::from("Zdravo!"));
    let sporocilo2 = Sporocilo::Stevilo(42);
    let sporocilo3 = Sporocilo::Kombinacija(String::from("Odgovor:"), 42);

    obdelaj_sporocilo(sporocilo1);
    obdelaj_sporocilo(sporocilo2);
    obdelaj_sporocilo(sporocilo3);
}
```

Izhod tega programa bi bil:

```
Prejel sem besedilo: Zdravo!
Prejel sem število: 42
Prejel sem kombinacijo: Odgovor: in 42
```

### Zaključek

**Pattern Matching** v Rustu je zelo močno orodje, ki omogoča enostavno obdelavo različnih vrst podatkov, shranjenih v enumeracijah. Z uporabo `match` lahko preverimo, katera varianta je trenutno uporabljena, in dostopimo do njenih vrednosti.

Če imaš kakršnakoli dodatna vprašanja ali potrebuješ dodatne razlage, le vprašaj! 😊
