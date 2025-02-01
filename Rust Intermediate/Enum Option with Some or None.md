Tukaj uporabljamo enum `Option`, ki vključuje `Some` in `None`.
### Kaj je `Option`?

Enum je `Option` enum, ki se uporablja za predstavljanje vrednosti, ki lahko obstaja (`Some`) ali pa ne obstaja (`None`). To je zelo uporabno, ko imamo opravka z vrednostmi, ki so lahko `null` ali `undefined` v drugih jezikih, vendar Rust ne dovoli neposredne uporabe `null` zaradi varnosti.

### Definicija `Option`

`Option` je definiran v standardni knjižnici Rusta takole:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

- `Some(T)` pomeni, da obstaja vrednost tipa `T`.
- `None` pomeni, da vrednost ne obstaja.

### Primer uporabe `Option`

Poglejmo si primer, kako bi uporabili `Option` v praksi.

#### Primer 1: Preprosta funkcija, ki vrne `Option`

Recimo, da imamo funkcijo, ki poišče število v seznamu in vrne njegov indeks. Če število ni v seznamu, vrne `None`.

```rust
fn poisci_indeks(seznam: Vec<i32>, iskano: i32) -> Option<usize> {
    for (indeks, &vrednost) in seznam.iter().enumerate() {
        if vrednost == iskano {
            return Some(indeks);
        }
    }
    None
}

fn main() {
    let seznam = vec![10, 20, 30, 40, 50];
    let iskano = 30;

    match poisci_indeks(seznam, iskano) {
        Some(indeks) => println!("Število {} najdeno na indeksu {}", iskano, indeks),
        None => println!("Število {} ni bilo najdeno v seznamu.", iskano),
    }
}
```

V tem primeru:
- Če je število `30` v seznamu, bo funkcija vrnila `Some(indeks)`, kjer je `indeks` položaj števila v seznamu.
- Če števila ni v seznamu, bo funkcija vrnila `None`.

#### Primer 2: Uporaba `Option` z `match`

`match` je močan mehanizem v Rustu, ki omogoča obravnavo različnih možnosti.

```rust
fn deljenje(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None
    } else {
        Some(a / b)
    }
}

fn main() {
    let a = 10.0;
    let b = 2.0;

    match deljenje(a, b) {
        Some(rezultat) => println!("Rezultat deljenja je {}", rezultat),
        None => println!("Deljenje z nič ni dovoljeno!"),
    }
}
```

V tem primeru:
- Če `b` ni `0.0`, bo funkcija vrnila `Some(rezultat)`, kjer je `rezultat` rezultat deljenja.
- Če je `b` enak `0.0`, bo funkcija vrnila `None`, ker deljenje z nič ni dovoljeno.

### Uporaba `unwrap` in `expect`

Včasih želimo direktno dobiti vrednost iz `Option`, vendar moramo biti previdni, ker lahko to povzroči napako, če je `Option` enak `None`.

- `unwrap()`: Vzame vrednost iz `Some`, če je `Option` enak `None`, pa povzroči paniko (program se sesuje).

```rust
let x = Some(5);
println!("{}", x.unwrap()); // Izpiše: 5

let y: Option<i32> = None;
println!("{}", y.unwrap()); // Panika!
```

- `expect()`: Podobno kot `unwrap()`, vendar omogoča, da podamo sporočilo, ki se izpiše ob paniki.

```rust
let x = Some(5);
println!("{}", x.expect("Vrednost ne obstaja!")); // Izpiše: 5

let y: Option<i32> = None;
println!("{}", y.expect("Vrednost ne obstaja!")); // Panika s sporočilom "Vrednost ne obstaja!"
```

### Zaključek

`Option` je zelo uporaben enum v Rustu, ki nam pomaga obvladati primere, ko vrednost lahko obstaja ali pa ne. Z uporabo `Some` in `None` lahko varno delamo z možnimi `null` vrednostmi, ne da bi se srečali z napakami, ki so pogoste v drugih jezikih.

Upam, da ti je ta razlaga pomagala razumeti, kako uporabljati `Option` v Rustu! Če imaš kakšna vprašanja, lahko vprašaš. 😊
