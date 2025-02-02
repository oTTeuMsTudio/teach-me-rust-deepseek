Danes razumevamo, kako delujejo **lifetimes dobe**, **generiki** in **strukture (structs)** v nizkonivojskem programskem jeziku Rust.
### 1. Čas veljavnosti spremenljivke ali reference (Lifetimes)

V Rustu se **`lifetime` doba** nanaša na to, kako dolgo je neka spremenljivka ali referenca veljavna. Rust uporablja `lifetime` dobo, da zagotovi, da reference nikoli ne kažejo na neveljavne podatke.

#### Primer:

```rust
fn daljsi_niz<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

- `'a` je oznaka za življenjsko dobo. Pove, da bosta `x` in `y` imeli enako življenjsko dobo.
- Funkcija vrne referenco na daljši niz, ki ima isto življenjsko dobo kot vhodni nizi.

### 2. Generiki

Generiki omogočajo, da napišemo kodo, ki deluje z različnimi tipi podatkov. To je koristno, ko želimo, da naša funkcija ali struktura deluje na splošno, ne glede na konkretni tip.

#### Primer:

```rust
fn najvecji<T: PartialOrd>(a: T, b: T) -> T {
    if a > b {
        a
    } else {
        b
    }
}
```

- `T` je generični tip. `PartialOrd` je lastnost (trait), ki omogoča primerjavo vrednosti.
- Funkcija `najvecji` lahko sprejme katerikoli tip, ki implementira `PartialOrd`, in vrne večjo vrednost.

### 3. Strukture (Structs)

Strukture so uporabniško definirani tipi, ki združujejo več vrednosti v eno enoto. Strukture so podobne razredom v drugih jezikih, vendar nimajo metod (razen če jih eksplicitno dodamo).

#### Primer:

```rust
struct Oseba {
    ime: String,
    starost: u32,
}

impl Oseba {
    fn nov(ime: String, starost: u32) -> Oseba {
        Oseba { ime, starost }
    }

    fn pozdravi(&self) {
        println!("Pozdravljen, {}! Star si {} let.", self.ime, self.starost);
    }
}
```

- `Oseba` je struktura z dvema poljema: `ime` in `starost`.
- `impl` blok definira metode, ki jih lahko kličemo na instancah strukture `Oseba`.
- Metoda `nov` je konstruktor, ki ustvari novo instanco strukture.
- Metoda `pozdravi` izpiše pozdrav, ki vključuje ime in starost osebe.

### Kombinacija vseh treh konceptov

Zdaj pa poglejmo, kako lahko združimo vse tri koncepte: življenjske dobe, generike in strukture.

```rust
struct Oseba<'a, T> {
    ime: &'a str,
    starost: T,
}

impl<'a, T> Oseba<'a, T> {
    fn nov(ime: &'a str, starost: T) -> Oseba<'a, T> {
        Oseba { ime, starost }
    }

    fn pozdravi(&self) {
        println!("Pozdravljen, {}! Star si {:?} let.", self.ime, self.starost);
    }
}
```

- `Oseba` je zdaj generična struktura, ki sprejme življenjsko dobo `'a` in generični tip `T`.
- Polje `ime` je referenca na niz z življenjsko dobo `'a`.
- Polje `starost` je generičnega tipa `T`.
- Metoda `nov` ustvari novo instanco strukture `Oseba`.
- Metoda `pozdravi` izpiše pozdrav, ki vključuje ime in starost osebe.

### Zaključek

- **Življenjska doba (`'a`)**: Zagotavlja, da reference ostanejo veljavne.
- **Generiki (`T`)**: Omogočajo, da koda deluje z različnimi tipi.
- **Strukture (`struct`)**: Združujejo podatke v eno enoto.

Upam, da ti je to razlago koristno! Če imaš kakršnakoli vprašanja, jih lahko zastaviš. 😊
