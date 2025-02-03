### Kaj je `HashMap`?

`HashMap` je zbirka (collection), ki shranjuje pare ključ-vrednost (key-value pairs). Vsak ključ je edinstven in se uporablja za dostop do pripadajoče vrednosti. `HashMap` uporablja **hash funkcijo** za preslikavo ključev v njihove pripadajoče vrednosti, kar omogoča hitro iskanje, vstavljanje in brisanje.

### Kaj je `HashSet`?

`HashSet` je zbirka, ki shranjuje samo edinstvene vrednosti (brez ključev). Prav tako uporablja hash funkcijo za hitro iskanje in preverjanje, ali element obstaja v množici. `HashSet` je podoben matematični množici, kjer vsak element obstaja samo enkrat.

### Uporaba `HashMap` v Rustu

Najprej moramo uvoziti `HashMap` iz standardne knjižnice:

```rust
use std::collections::HashMap;
```

#### Ustvarjanje `HashMap`

```rust
let mut map = HashMap::new();
```

#### Dodajanje elementov

```rust
map.insert(String::from("ključ1"), 10);
map.insert(String::from("ključ2"), 20);
```

#### Dostop do vrednosti

```rust
if let Some(vrednost) = map.get("ključ1") {
    println!("Vrednost za 'ključ1': {}", vrednost);
}
```

#### Posodabljanje vrednosti

```rust
map.insert(String::from("ključ1"), 30); // Posodobi vrednost za "ključ1"
```

#### Brisanje elementov

```rust
map.remove("ključ2");
```

#### Iteracija čez `HashMap`

```rust
for (ključ, vrednost) in &map {
    println!("{}: {}", ključ, vrednost);
}
```

### Uporaba `HashSet` v Rustu

Najprej uvozimo `HashSet` iz standardne knjižnice:

```rust
use std::collections::HashSet;
```

#### Ustvarjanje `HashSet`

```rust
let mut set = HashSet::new();
```

#### Dodajanje elementov

```rust
set.insert("jabolko");
set.insert("banana");
set.insert("jabolko"); // Ta vnos ne bo dodan, ker "jabolko" že obstaja
```

#### Preverjanje obstoja elementa

```rust
if set.contains("banana") {
    println!("Banana je v množici!");
}
```

#### Brisanje elementov

```rust
set.remove("banana");
```

#### Iteracija čez `HashSet`

```rust
for element in &set {
    println!("{}", element);
}
```

### Primerjava `HashMap` in `HashSet`

- **`HashMap`** se uporablja, ko želimo shranjevati pare ključ-vrednost.
- **`HashSet`** se uporablja, ko želimo shranjevati samo edinstvene vrednosti.

### Zaključek

`HashMap` in `HashSet` sta zelo uporabni podatkovni strukturi v Rustu, ki omogočata hitro iskanje in manipulacijo s podatki. `HashMap` je idealen za situacije, kjer potrebujemo povezavo med ključi in vrednostmi, medtem ko `HashSet` uporabljamo, ko nas zanimajo samo edinstvene vrednosti.
