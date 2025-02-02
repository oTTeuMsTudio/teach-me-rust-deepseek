### 1. **Polimorfizem**
Polimorfizem pomeni, da lahko ista koda deluje z različnimi tipi podatkov. Tukaj to dosežemo z uporabo lastnosti (traits) in generičnih tipov (generics).

### 2. **Lastnosti (Traits)**
Lastnosti (traits) so podobne vmesnikom (interfaces) v drugih jezikih. Definirajo skupno vedenje, ki ga lahko implementirajo različni tipi.

#### Primer:
```rust
trait Pozdravi {
    fn pozdravi(&self) -> String;
}

struct Clovek {
    ime: String,
}

struct Pes {
    ime: String,
}

impl Pozdravi for Clovek {
    fn pozdravi(&self) -> String {
        format!("Zdravo, jaz sem {}.", self.ime)
    }
}

impl Pozdravi for Pes {
    fn pozdravi(&self) -> String {
        format!("Hov, jaz sem pes {}.", self.ime)
    }
}
```

Tukaj imamo lastnost `Pozdravi`, ki jo implementirata `Clovek` in `Pes`. Oba tipa imata metodo `pozdravi`, ki vrne niz.

### 3. **Generični tipi (Generics)**
Generični tipi omogočajo, da napišemo kodo, ki deluje s poljubnimi tipi podatkov. To je še posebej uporabno, ko želimo, da naša koda deluje z različnimi podatki, ne da bi jo morali pisati večkrat.

#### Primer:
```rust
fn izpisi_pozdrav<T: Pozdravi>(objekt: T) {
    println!("{}", objekt.pozdravi());
}
```

Tukaj je `T` generični tip, ki mora implementirati lastnost `Pozdravi`. Funkcija `izpisi_pozdrav` lahko sprejme katerikoli tip, ki implementira `Pozdravi`.

### 4. **Uporaba skupaj**
Zdaj pa združimo vse skupaj:

```rust
fn main() {
    let clovek = Clovek { ime: String::from("truenjenir") };
    let pes = Pes { ime: String::from("oTTeuM") };

    izpisi_pozdrav(clovek);
    izpisi_pozdrav(pes);
}
```

Napisano bo:
```
Zdravo, jaz sem truenjenir.
Hov, jaz sem pes oTTeuM.
```

### 5. **Dinamični polimorfizem**
Včasih želimo, da lahko tip objekta določimo med izvajanjem programa (runtime). To dosežemo z uporabo lastnosti in kazalcev (`&dyn`).

#### Primer:
```rust
fn main() {
    let clovek = Clovek { ime: String::from("truenjenir") };
    let pes = Pes { ime: String::from("oTTeuM") };

    let objekti: Vec<&dyn Pozdravi> = vec![&clovek, &pes];

    for objekt in objekti {
        println!("{}", objekt.pozdravi());
    }
}
```

Tukaj imamo vektor `objekti`, ki vsebuje reference na objekte, ki implementirajo lastnost `Pozdravi`. Med izvajanjem programa lahko določimo, kateri tipi bodo v vektorju.

### 6. **Zaključek**
- **Lastnosti (Traits)** definirajo skupno vedenje.
- **Generični tipi (Generics)** omogočajo, da koda deluje z različnimi tipi.
- **Polimorfizem** omogoča, da ista koda deluje z različnimi tipi podatkov.

