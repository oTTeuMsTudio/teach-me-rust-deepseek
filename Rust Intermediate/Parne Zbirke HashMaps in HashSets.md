### Kaj je `HashMap`?

`HashMap` je zbirka (collection), ki shranjuje pare ključ-vrednost (key-value pairs). Vsak ključ je unikat in brez njega nimamo dostopa do pripadajoče vrednosti. `HashMap` uporablja **hash funkcijo**, s katero preslika ključ v pripadajoče vrednosti, kar omogoča hitro iskanje, vstavljanje in brisanje.

### Kaj je `HashSet`?

`HashSet` je zbirka, ki shranjuje samo unikatne vrednosti in to brez ključev. Prav tako uporablja hash funkcijo za hitro iskanje in preverjanje, ali element obstaja v množici ali ne. `HashSet` je podoben matematični množici, kjer vsak element obstaja samo enkrat.

### Uporaba `HashMap` v Rustu

Najprej moramo uvoziti `HashMap` iz standardne knjižnice:

```rust
use std::collections::HashMap;
```

#### Ustvarimo `HashMap`

```rust
let mut map = HashMap::new();
```

#### Dodamo element

```rust
map.insert(String::from("ključ1"), 10);
map.insert(String::from("ključ2"), 20);
```

#### Dostopimo do vrednosti

```rust
if let Some(vrednost) = map.get("ključ1") {
    println!("Vrednost za 'ključ1': {vrednost}");
}
```

#### Posodobimo vrednost

```rust
map.insert(String::from("ključ1"), 30); // Posodobi vrednost za "ključ1"
```

#### Brišemo element

```rust
map.remove("ključ2");
```

#### Iteriramo čez `HashMap`

```rust
for (ključ, vrednost) in &map {
    println!("{kljuc}: {vrednost}");
}
```

### Uporaba `HashSet` v Rustu

Najprej uvozimo `HashSet` iz standardne knjižnice:

```rust
use std::collections::HashSet;
```

#### Ustvarimo `HashSet`

```rust
let mut set = HashSet::new();
```

#### Dodamo element

```rust
set.insert("jabolko");
set.insert("banana");
set.insert("jabolko"); // Ta vnos ne bo dodan, ker "jabolko" že obstaja
```

#### Preverimo element

```rust
if set.contains("banana") {
    println!("Banana je v množici!");
}
```

#### Brišemo element

```rust
set.remove("banana");
```

#### Iteriramo čez `HashSet`

```rust
for element in &set {
    println!("{}", element);
}
```

### Primerjamo `HashMap` in `HashSet`

- **`HashMap`** se uporablja, ko želimo shranjevati pare ključ-vrednost.
- **`HashSet`** se uporablja, ko želimo shranjevati samo edinstvene vrednosti.

### Zaključek

`HashMap` in `HashSet` sta zelo uporabni podatkovni strukturi v Rustu, ki omogočata hitro iskanje in manipulacijo s podatki. `HashMap` je idealen za situacije, kjer potrebujemo povezavo med ključi in vrednostmi, medtem ko `HashSet` uporabljamo, ko nas zanimajo samo edinstvene vrednosti.
