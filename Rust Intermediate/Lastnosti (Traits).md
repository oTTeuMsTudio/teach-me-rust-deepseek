### Kaj so lastnosti (traits)?
**Lastnosti (traits)** so podobni vmesnikom (interfaces) v drugih jezikih, kot so Java ali C#. Omogočajo definiranje skupnega vedenja, ki ga lahko implementirajo različni tipi. Lastnosti so osnova za abstrakcijo in polimorfizem.

### Kako definiramo lastnost?
Lastnost definiramo s ključno besedo `trait`. Znotraj lastnosti lahko definiramo metode (funkcije), ki jih morajo implementirati tipi, ki bodo uporabljali to lastnost.

Primer:
```rust
trait Pozdrav {
    fn pozdravi(&self) -> String;
}
```
Tukaj smo definirali lastnost `Pozdrav`, ki zahteva metodo `pozdravi`, ki vrne niz (`String`).

### Kako implementiramo lastnost?
Lastnost implementiramo za določen tip z uporabo ključne besede `impl`.

Primer:
```rust
struct Oseba {
    ime: String,
}

impl Pozdrav for Oseba {
    fn pozdravi(&self) -> String {
        format!("Zdravo, {}!", self.ime)
    }
}
```
Tukaj smo implementirali lastnost `Pozdrav` za tip `Oseba`. Metoda `pozdravi` zdaj vrne pozdrav, ki vključuje ime osebe.

### Kako uporabimo lastnost?
Lastnosti lahko uporabimo kot omejitve pri generičnih tipih ali pri definiranju funkcij, ki sprejemajo različne tipe, ki implementirajo določeno lastnost.

Primer:
```rust
fn izpisi_pozdrav<T: Pozdrav>(objekt: T) {
    println!("{}", objekt.pozdravi());
}

fn main() {
    let oseba = Oseba { ime: String::from("Ana") };
    izpisi_pozdrav(oseba);
}
```
V tem primeru funkcija `izpisi_pozdrav` sprejme katerikoli tip, ki implementira lastnost `Pozdrav`, in izpiše pozdrav.

### Privzete metode v lastnostih
Lastnosti lahko vsebujejo tudi **privzete metode**, ki jih tipi lahko prekrijejo (override), če želijo.

Primer:
```rust
trait Pozdrav {
    fn pozdravi(&self) -> String {
        String::from("Zdravo!")
    }
}

impl Pozdrav for Oseba {
    // Tukaj ne potrebujemo implementirati `pozdravi`, ker uporabljamo privzeto metodo.
}
```

### Več lastnosti
Tip lahko implementira več lastnosti. To storimo tako, da ločimo lastnosti z `+`.

Primer:
```rust
trait Pozdrav {
    fn pozdravi(&self) -> String;
}

trait Opis {
    fn opis(&self) -> String;
}

impl Pozdrav for Oseba {
    fn pozdravi(&self) -> String {
        format!("Zdravo, {}!", self.ime)
    }
}

impl Opis for Oseba {
    fn opis(&self) -> String {
        format!("To je oseba z imenom {}.", self.ime)
    }
}

fn izpisi_podatke<T: Pozdrav + Opis>(objekt: T) {
    println!("{} {}", objekt.pozdravi(), objekt.opis());
}
```

### Zaključek
Lastnosti (traits) so močno orodje v Rustu, ki omogoča definiranje skupnega vedenja za različne tipe. Uporabljajo se za abstrakcijo, polimorfizem in izogibanje podvajanju kode.

Če imaš kakršnakoli dodatna vprašanja ali potrebuješ dodatne razlage, le vprašaj! 😊
