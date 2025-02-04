Kako napisati preprosto igro ugibanja?
### 1. Namestitev Rusta
Če še nimaš nameščenega Rusta, ga lahko namestiš s pomočjo `rustup`. Odpri terminal in vnesi naslednjo ukaz:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Po namestitvi lahko preveriš, ali je Rust pravilno nameščen:

```bash
rustc --version
```

### 2. Ustvarjanje novega projekta
Ustvarimo nov projekt z imenom `ugibanje`:

```bash
cargo new ugibanje
cd ugibanje
```

### 3. Pisanje kode
Odpri datoteko `src/main.rs` v svojem najljubšem urejevalniku kode. Vsebino datoteke zamenjaj z naslednjo kodo:

```rust
use std::io;
use rand::Rng;
use std::cmp::Ordering;

fn main() {
    println!("Ugani število!");

    let skrivno_stevilo = rand::rng().random_range(1..101);

    loop {
        println!("Prosim, vnesi svoje ugibanje.");

        let mut ugibanje = String::new();

        io::stdin()
            .read_line(&mut ugibanje)
            .expect("Napaka pri branju vnosa");

        let ugibanje: u32 = match ugibanje.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("Prosim, vnesi veljavno število!");
                continue;
            }
        };

        println!("Ugibal si: {}", ugibanje);

        match ugibanje.cmp(&skrivno_stevilo) {
            Ordering::Less => println!("Pre majhno!"),
            Ordering::Greater => println!("Pre veliko!"),
            Ordering::Equal => {
                println!("Točno! Zmagal si!");
                break;
            }
        }
    }
}
```

### 4. Razlaga kode

- **Uvoz knjižnic**: Uvozimo potrebne knjižnice za branje vnosa (`io`), generiranje naključnih števil (`rand::Rng`) in primerjanje (`std::cmp::Ordering`).

- **Generiranje skrivnega števila**: Uporabimo `rand::rng().random_range(1..101)` za generiranje naključnega števila med 1 in 100.

- **Zanka za ugibanje**: Uporabimo neskončno zanko (`loop`), ki se bo izvajala, dokler igralec ne ugane pravilnega števila.

- **Branje vnosa**: Uporabimo `io::stdin().read_line(&mut ugibanje)` za branje vnosa igralca. Vnos pretvorimo v število (`u32`) in obravnavamo morebitne napake.

- **Primerjanje ugibanja**: Uporabimo `match` za primerjanje ugibanja s skrivnim številom. Če je ugibanje preveliko ali premajhno, izpišemo ustrezno sporočilo. Če je ugibanje pravilno, igralcu čestitamo in končamo zanko.

### 5. Dodajanje odvisnosti
Da bi lahko uporabili knjižnico `rand`, jo moramo dodati v `Cargo.toml`:

```toml
[dependencies]
rand = "0.8"
```

### 6. Zagon igre
Ko je koda pripravljena, lahko igro zaženemo z naslednjim ukazom:

```bash
cargo run
```

### 7. Igrajmo se!
Igralec bo moral ugibati število, dokler ne ugane pravilnega. Program bo igralcu dajal napotke, ali je njegovo ugibanje preveliko ali premajhno.

