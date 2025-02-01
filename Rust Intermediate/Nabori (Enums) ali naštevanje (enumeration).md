

### `Enums` (nabori, nabor, naštevanje, enumaracija)

`Enum` je kratica za "enumeration" (naštevanje, nabor) in predstavlja tip podatkov, ki omogoča definiranje, naštevanje v množici vrednosti. V Rust-u lahko z `enum`-si definiraš tip, ki ima več možnih variant. Vsaka varianta lahko vsebuje različne podatke ali pa je lahko tudi prazna.

### Osnovna sintaksa

Osnovna sintaksa za definiranje `enum`-ov v Rust-u je naslednja:

```rust
enum EnumIme {
    Varianta1,
    Varianta2,
    Varianta3,
}
```

Tukaj je `ImeEnum` ime nabora, `Varianta1`, `Varianta2` in `Varianta3` pa so možne vrednosti, ki jih lahko sprejme spremenljivka tega tipa.

### Primer: Barve

Poglejmo si preprost primer, kjer definiramo `enum` za barve:

```rust
enum Barva {
    Rdeca,
    Zelena,
    Modra,
}
```

Tukaj smo definirali `enum` z imenom `Barva`, ki ima tri variante: `Rdeča`, `Zelena` in `Modra`.

### Uporaba `enum`

Ko imamo definiran `enum`, ga lahko uporabimo za ustvarjanje spremenljivk:

```rust
let moja_barva = Barva::Rdeca;
```

Tukaj smo ustvarili spremenljivko `moja_barva`, ki ima vrednost `Barva::Rdeca`.

### `enum` s podatki

V Rustu lahko vsaka varianta `enum` vsebuje tudi dodatne podatke. Na primer, definirajmo `enum` za geometrijske oblike:

```rust
enum Model {
    Krog(f64),          // Krog s polmerom
    Pravokotnik(f64, f64), // Pravokotnik s širino in višino
    Kvadrat(f64),       // Kvadrat s stranico
}
```

Tukaj ima vsaka varianta v `enum` `Model` svoje podatke:

- `Krog` vsebuje polmer tipa `f64`.
- `Pravokotnik` vsebuje širino in višino tipa `f64`.
- `Kvadrat` vsebuje dolžino stranice tipa `f64`.

### Uporaba `enum`-a s podatki

Uporaba takšnega `enum`-a izgleda takole:

```rust
let krog = Model::Krog(5.0);
let pravokotnik = Model::Pravokotnik(10.0, 20.0);
let kvadrat = Model::Kvadrat(15.0);
```

### Obdelava `enum`-a z `match`

Pogosto želimo obdelati različne variante `enum`-a z uporabo `match` izraza. Na primer:

```rust
fn izracunaj_ploscino(model: Model) -> f64 {
    match model {
        Model::Krog(r) => 3.14 * r * r,
        Model::Pravokotnik(w, h) => w * h,
        Model::Kvadrat(s) => s * s,
    }
}
```

Tukaj funkcija `izracunaj_ploscino` sprejme `Model` in z `match` izrazom obdela vsako varianto, da izračuna ploščino.

### Zaključek

`Enum` v Rustu je zelo močno orodje, ki omogoča definiranje tipov z več možnimi vrednostmi. Lahko vsebujejo tudi dodatne podatke, kar jih naredi še bolj fleksibilne. Z uporabo `match`-a lahko enostavno obdelujemo različne variante `enum`-a.
