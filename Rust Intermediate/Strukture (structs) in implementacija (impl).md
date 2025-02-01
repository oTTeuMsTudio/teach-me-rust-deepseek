Rust je močan in varen sistemski programski jezik, ki omogoča uporabo struktur (`struct`) za organizacijo podatkov. Poleg tega lahko z uporabo implementacije `impl` definiramo metode, ki delujejo na teh strukturah.

### 1. Structure `struct`

Struktura (`struct`) je zbirka podatkov, ki jih lahko združimo v eno entiteto. Vsak podatek v strukturi ima svoje ime in tip.

Primer:

```rust
struct Oseba {
    ime: String,
    starost: u32,
}
```

Tukaj smo definirali strukturo `Oseba`, ki ima dva polja: `ime` tipa `String` in `starost` tipa `u32`.

### 2. Ustvarjanje instance strukture

Ko imamo strukturo, lahko ustvarimo njeno instanco (objekt).

```rust
let oseba = Oseba {
    ime: String::from("Janez"),
    starost: 30,
};
```

Tukaj smo ustvarili instanco `Oseba` z imenom "Janez" in starostjo 30.

### 3. Dostop do polj strukture

Do posameznih polj strukture lahko dostopamo z uporabo pike (`.`).

```rust
println!("Ime: {}", oseba.ime);
println!("Starost: {}", oseba.starost);
```

### 4. Definiranje metode z `impl`

Z `impl` lahko definiramo metode, ki delujejo na strukturah. Metode so funkcije, ki so povezane s strukturo.

Primer:

```rust
impl Oseba {
    fn pozdravi(&self) {
        println!("Pozdravljen, {}!", self.ime);
    }

    fn je_odrasel(&self) -> bool {
        self.starost >= 18
    }
}
```

Tukaj smo definirali dve metodi:

- `pozdravi`: Izpiše pozdrav z imenom osebe.
- `je_odrasel`: Vrne `true`, če je oseba stara 18 let ali več, sicer `false`.

### 5. Klic metod

Metode lahko kličemo na instancah strukture.

```rust
let oseba = Oseba {
    ime: String::from("Ana"),
    starost: 25,
};

oseba.pozdravi(); // Izpiše: Pozdravljena, Ana!
println!("Je odrasel? {}", oseba.je_odrasel()); // Izpiše: Je odrasel? true
```

### 6. Asociirane funkcije

Poleg metod lahko v `impl` bloku definiramo tudi asociirane funkcije, ki niso vezane na instanco strukture. Te funkcije se običajno uporabljajo za ustvarjanje novih instanc.

Primer:

```rust
impl Oseba {
    fn nova(ime: String, starost: u32) -> Oseba {
        Oseba { ime, starost }
    }
}

let oseba = Oseba::nova(String::from("Miha"), 22);
```

Tukaj smo definirali asociirano funkcijo `nova`, ki ustvari novo instanco `Oseba`.

### 7. Zaključek

Strukture (`struct`) in metode (`impl`) so osnovni gradniki Rust-ovih programov. Omogočajo nam, da organiziramo podatke in funkcionalnost na smiseln način. Z uporabo `impl` lahko definiramo metode, ki delujejo na strukturah, in asociirane funkcije za ustvarjanje novih instanc.
