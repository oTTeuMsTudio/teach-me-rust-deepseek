Delo z low-level (nizkonivojskimi) binarnimi podatki v Rustu je zanimiva tema. Rust omogoča direktno manipulacijo s pomnilnikom in binarnimi podatki, hkrati pa zagotavlja varnost. V tej lekciji se bomo osredotočili na osnove dela z binarnimi podatki v Rust-u.

### 1. Osnove binarnih podatkov

Binarni podatki so zapisani v obliki bitov (0 in 1). V Rust-u lahko delamo z binarnimi podatki z uporabo različnih tipov, kot so `u8`, `u16`, `u32`, `u64`, itd., ki predstavljajo nepredznačene cele številke različnih velikosti.

### 2. Delo z bajti (`u8`)

Najosnovnejša enota binarnih podatkov je bajt, ki ga v Rust-u predstavlja tip `u8`. Bajt je 8-bitno število, ki lahko zavzame vrednosti od 0 do 255.

```rust
fn main() {
    let bajt: u8 = 0b10101010; // Binarni zapis števila
    println!("Bajt: {:b}", bajt); // Izpis v binarni obliki
}
```

### 3. Bitne operacije

Rust podpira različne bitne operacije, kot so AND (`&`), OR (`|`), XOR (`^`), shift (`<<`, `>>`) in NOT (`!`).

```rust
fn main() {
    let a: u8 = 0b1100;
    let b: u8 = 0b1010;

    println!("AND: {:b}", a & b); // 0b1000
    println!("OR: {:b}", a | b);  // 0b1110
    println!("XOR: {:b}", a ^ b); // 0b0110
    println!("Shift levo: {:b}", a << 1); // 0b11000
    println!("Shift desno: {:b}", a >> 1); // 0b0110
    println!("NOT: {:b}", !a); // 0b11110011 (odvisno od velikosti tipa)
}
```

### 4. Delo z več bajti

Če želimo delati z več bajti hkrati, lahko uporabimo večje tipe, kot so `u16`, `u32`, `u64`, ali pa uporabimo polja (`array`) ali slice-e.

```rust
fn main() {
    let podatki: [u8; 4] = [0b10101010, 0b11110000, 0b00001111, 0b11001100];

    for bajt in &podatki {
        println!("Bajt: {:b}", bajt);
    }
}
```

### 5. Pretvorbe med tipi

Pogosto bomo morali pretvarjati med različnimi tipi. Rust omogoča enostavne pretvorbe z uporabo `as`.

```rust
fn main() {
    let veliko_stevilo: u32 = 0x12345678;
    let bajt: u8 = (veliko_stevilo >> 24) as u8; // Vzamemo najvišji bajt

    println!("Najvišji bajt: {:x}", bajt); // Izpis v šestnajstiškem zapisu
}
```

### 6. Delo z bitnimi maskami

Bitne maske so uporabne za izolacijo specifičnih bitov v številu.

```rust
fn main() {
    let stevilo: u8 = 0b10101010;
    let maska: u8 = 0b00001111;

    let spodnji_biti = stevilo & maska; // Izoliramo spodnje 4 bite
    println!("Spodnji biti: {:b}", spodnji_biti); // 0b1010
}
```

### 7. Pakiranje in razpakiranje podatkov

V sistemih z nizkimi nivoji pogosto pakiramo več podatkov v eno število. Na primer, lahko pakiramo 4 2-bitne vrednosti v en `u8`.

```rust
fn main() {
    let a: u8 = 0b01;
    let b: u8 = 0b10;
    let c: u8 = 0b11;
    let d: u8 = 0b00;

    let pakirano: u8 = (a << 6) | (b << 4) | (c << 2) | d;
    println!("Pakirano: {:b}", pakirano); // 0b01101100

    // Razpakiranje
    let a_razpakirano = (pakirano >> 6) & 0b11;
    let b_razpakirano = (pakirano >> 4) & 0b11;
    let c_razpakirano = (pakirano >> 2) & 0b11;
    let d_razpakirano = pakirano & 0b11;

    println!("A: {:b}, B: {:b}, C: {:b}, D: {:b}", a_razpakirano, b_razpakirano, c_razpakirano, d_razpakirano);
}
```

### 8. Nevarni bloki (`unsafe`)

Včasih boste morda morali delati direktno s pomnilnikom, kar v Rust-u zahteva uporabo `unsafe` blokov. To je napredna tema, zato bodite previdni.

```rust
fn main() {
    let mut stevilo: u32 = 0x12345678;

    unsafe {
        let kazalec = &mut stevilo as *mut u32;
        *kazalec = 0x87654321;
    }

    println!("Spremenjeno število: {:x}", stevilo);
}
```

### Zaključek

Delo z binarnimi podatki v Rust-u je močno orodje, ki omogoča nizkonivojsko optimizacijo in kontrolo nad podatki. Vendar pa je pomembno biti previden, saj lahko neprevidna manipulacija s pomnilnikom povzroči napake. Rust-ovi varnostni mehanizmi, kot so lastništvo in preverjanje meja, vam pomagajo preprečiti pogoste napake.
