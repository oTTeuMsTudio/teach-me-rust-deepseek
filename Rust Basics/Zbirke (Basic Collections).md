Rust ima močno podporo za zbirke (`Collections`). Zbirke so podatkovne strukture za shranjevanje in upravljanje vrednosti. Najosnovnejše zbirke v Rustu so `Vec`, `HashMap` in `HashSet`. 
### 1. `Vec` (Vektor)
`Vec` je dinamično polje, ki lahko raste ali se skrči. Shranjuje vrednosti v zaporedju in omogoča hitr dostop do elementov po indeksu.

#### Ustvarjanje vektorja
```rust
let mut vektor: Vec<i32> = Vec::new(); // Prazen vektor
vektor.push(1); // Dodajanje elementov
vektor.push(2);
vektor.push(3);
```

#### Dostop do elementov
```rust
let prvi_element = vektor[0]; // Dostop po indeksu
```

#### Iteracija čez vektor
```rust
for element in &vektor {
    println!("{}", element);
}
```

### 2. `HashMap` (Zemljevid)
`HashMap` shranjuje pare ključ-vrednost. Ključi so edinstveni in omogočajo hitro iskanje vrednosti.

#### Ustvarjanje zemljevida
```rust
use std::collections::HashMap;

let mut zemljevid = HashMap::new();
zemljevid.insert("ključ1", "vrednost1");
zemljevid.insert("ključ2", "vrednost2");
```

#### Dostop do vrednosti
```rust
if let Some(vrednost) = zemljevid.get("ključ1") {
    println!("Vrednost: {}", vrednost);
}
```

#### Iteracija čez zemljevid
```rust
for (ključ, vrednost) in &zemljevid {
    println!("{}: {}", ključ, vrednost);
}
```

### 3. `HashSet` (Množica)
`HashSet` je zbirka edinstvenih vrednosti brez vrstnega reda. Uporabno je, ko želimo zagotoviti, da so vsi elementi edinstveni.

#### Ustvarjanje množice
```rust
use std::collections::HashSet;

let mut množica = HashSet::new();
množica.insert(1);
množica.insert(2);
množica.insert(2); // Ta vnos ne bo dodan, ker je 2 že prisoten
```

#### Preverjanje prisotnosti
```rust
if množica.contains(&1) {
    println!("1 je v množici!");
}
```

#### Iteracija čez množico
```rust
for element in &množica {
    println!("{}", element);
}
```

### Zaključek
To so osnove zbirk v Rust-u. `Vec` je odličen za zaporedja, `HashMap` za pare ključ-vrednost in `HashSet` za edinstvene vrednosti. Rustova standardna knjižnica ponuja še več kolekcij, kot so `LinkedList`, `BTreeMap`, `BTreeSet` itd., vendar so `Vec`, `HashMap` in `HashSet` najpogosteje uporabljene.
