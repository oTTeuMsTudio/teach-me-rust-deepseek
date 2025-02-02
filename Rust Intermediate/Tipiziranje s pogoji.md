### 1. **Pattern Matching**
Pattern Matching je tipiziranje, ki ti omogoča, da preveriš in razčleniš podatke glede na njihovo strukturo. Uporablja se predvsem z ukazom `match`.

Primer:
```rust
let stevilo = 5;

match stevilo {
    1 => println!("Ena"),
    2 => println!("Dva"),
    3 | 4 | 5 => println!("Tri, štiri ali pet"),
    _ => println!("Nekaj drugega"),
}
```
- `match` preveri vrednost `število` in izvede kodo, ki ustreza prvemu ujemajočemu se vzorcu.
- `_` je "wildcard", ki ujema karkoli.

### 2. **Match Guards (Pogoji pri ujemanju)**
Match Guards so dodatni pogoji, ki jih lahko dodamo vzorcem v `match` izrazu. Uporabimo jih z ključno besedo `if`.

Primer:
```rust
let stevilo = Some(5);

match stevilo {
    Some(x) if x < 5 => println!("Manj kot pet"),
    Some(x) if x == 5 => println!("Točno pet"),
    Some(x) => println!("Več kot pet"),
    None => println!("Nič"),
}
```
- `if x < 5` je match guard, ki preveri dodatni pogoj.
- Če pogoj ni izpolnjen, se ta veja ne izvede.

### 3. **Structs (Strukture)**
Structs so uporabniško definirane podatkovne strukture, ki združujejo več vrednosti v eno enoto.

Primer:
```rust
struct Oseba {
    ime: String,
    starost: u8,
}

let oseba = Oseba {
    ime: String::from("Janez"),
    starost: 30,
};

match oseba {
    Oseba { ime, starost if starost >= 18 } => println!("{} je polnoleten.", ime),
    Oseba { ime, .. } => println!("{} je mladoletnik.", ime),
}
```
- `Oseba { ime, starost if starost >= 18 }` je vzorec, ki ujema strukturo `Oseba` in hkrati preveri pogoj `starost >= 18`.
- `..` ignorira preostala polja v strukturi.

### Skupni Primer
```rust
struct Tocka {
    x: i32,
    y: i32,
}

fn preveri_tocko(tocka: Tocka) {
    match tocka {
        Tocka { x, y } if x == y => println!("Točka je na diagonali."),
        Tocka { x, y } if x == 0 => println!("Točka je na y-osi."),
        Tocka { x, y } if y == 0 => println!("Točka je na x-osi."),
        Tocka { x, y } => println!("Točka je na ({}, {}).", x, y),
    }
}

fn main() {
    let tocka = Tocka { x: 3, y: 3 };
    preveri_tocko(tocka);
}
```
- Ta primer preveri, ali je točka na diagonali, x-osi ali y-osi, ali pa je na drugem mestu.

### Zaključek
- **Pattern Matching** ti omogoča, da preveriš in razčleniš podatke.
- **Match Guards** dodajo dodatne pogoje k vzorcem.
- **Structs** so podatkovne strukture, ki jih lahko uporabljaš v kombinaciji s pattern matchingom.

