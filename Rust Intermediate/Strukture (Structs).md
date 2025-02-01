Rust je varen sistemski programski jezik, ki omogoča uporabo struktur (`structs`) za organizacijo podatkov. Omogočajo ti, da združiš več vrednosti v eno samo enoto. 

### 1. Osnovna Struktura

Struktura je podobna "razredu" v drugih jezikih, vendar je bolj preprosta. Definirana je s ključno besedo `struct`.

```rust
struct Oseba {
    ime: String,
    starost: u32,
}
```

Tukaj smo definirali strukturo `Oseba`, ki ima dva polja:
- `ime` tipa `String` (niz),
- `starost` tipa `u32` (32-bitno nepredznačeno celo število).

### 2. Instanciranje Strukture

Ko imamo strukturo, lahko ustvarimo njeno instanco (objekt).

```rust
let oseba = Oseba {
    ime: String::from("Janez"),
    starost: 30,
};
```

Tukaj smo ustvarili instanco `Oseba` z imenom "Janez" in starostjo 30.

### 3. Dostop do Polj

Do polj strukture dostopamo s piko (`.`).

```rust
println!("Ime: {}", oseba.ime);
println!("Starost: {}", oseba.starost);
```

To bo izpisalo:
```
Ime: Janez
Starost: 30
```

### 4. Mutable Strukture

Če želimo spremeniti vrednosti polj, moramo instanco narediti mutable (spremenljivo) s ključno besedo `mut`.

```rust
let mut oseba = Oseba {
    ime: String::from("Janez"),
    starost: 30,
};

oseba.starost = 31;  // Spremenimo starost
```

### 5. Metode v Strukturah

Strukture lahko imajo tudi metode. Metode so funkcije, ki so vezane na strukturo. Definirane so znotraj bloka `impl`.

```rust
impl Oseba {
    fn pozdravi(&self) {
        println!("Pozdravljen, {}! Star si {} let.", self.ime, self.starost);
    }
}
```

Metodo `pozdravi` lahko pokličemo na instanci strukture:

```rust
oseba.pozdravi();  // Izpiše: Pozdravljen, Janez! Star si 31 let.
```

### 6. Povezane Funkcije

Poleg metod lahko definiramo tudi "povezane funkcije" (associated functions), ki niso vezane na instanco strukture. Te so podobne statičnim metodam v drugih jezikih.

```rust
impl Oseba {
    fn nova(ime: String, starost: u32) -> Oseba {
        Oseba { ime, starost }
    }
}
```

Uporaba:

```rust
let oseba = Oseba::nova(String::from("Ana"), 25);
```

### 7. Tuple Strukture

Rust ima tudi "tuple strukture", ki so podobne običajnim strukturam, vendar nimajo poimenovanih polj.

```rust
struct Barva(u8, u8, u8);

let rdeca = Barva(255, 0, 0);
```

Dostop do polj je prek indeksov:

```rust
println!("Rdeča: {}", rdeca.0);  // Izpiše: Rdeča: 255
```

### 8. Unit Strukture

Unit strukture (`unit structs`) so strukture brez polj. Uporabne so, ko želimo definirati tip, ki nima podatkov.

```rust
struct Prazno;

let p = Prazno;
```

### Zaključek

Strukture so orodje za organizacijo podatkov. Omogočajo ti, da združiš povezane podatke v eno enoto, kar olajša upravljanje in razumevanje kode. Z metodami in povezanimi funkcijami lahko strukturam dodajaš tudi funkcionalnost.
