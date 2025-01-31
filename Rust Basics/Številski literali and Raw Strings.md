### Številski literali v Rust-u

Številski literali so načini, kako v Rust-u zapisujemo števila. Rust podpira različne tipe števil, kot so cela števila (`integers`), plavajoča številka (`floats`) in števila v različnih številskih sistemih (binarni, osmiški, šestnajstiški).

#### 1. Cela števila (Integers)
Cela števila so števila brez decimalnih vejic. Rust podpira različne velikosti in predznake:

- `i8`, `i16`, `i32`, `i64`, `i128`, `isize` – predznačena cela števila (lahko so negativna).
- `u8`, `u16`, `u32`, `u64`, `u128`, `usize` – nepredznačena cela števila (samo pozitivna).

Primeri:
```rust
let x = 42; // i32 (privzeto)
let y: u64 = 100_000; // eksplicitno določen tip u64
let z = 0xff; // šestnajstiško število (255 v decimalnem sistemu)
let w = 0o77; // osmiško število (63 v decimalnem sistemu)
let v = 0b1111_0000; // binarno število (240 v decimalnem sistemu)
```

#### 2. Plavajoča številka (Floats)
Plavajoča številka so števila z decimalno vejico. Rust podpira dve velikosti:

- `f32` – 32-bitna plavajoča številka.
- `f64` – 64-bitna plavajoča številka (privzeto).

Primeri:
```rust
let a = 3.14; // f64 (privzeto)
let b: f32 = 2.718; // eksplicitno določen tip f32
let c = 6.022e23; // znanstveni zapis (6.022 * 10^23)
```

#### 3. Ločila za lažje branje
V Rust-u lahko uporabljate podčrtaje (`_`) za ločevanje števk, da so številke bolj berljive:

```rust
let milijon = 1_000_000; // 1000000
let pi = 3.141_592_653_59; // 3.14159265359
```

---

### Raw Strings v Rust-u

Raw strings so način zapisa nizov, kjer se ne uporablja escape zapis (kot je `\n` za novo vrstico ali `\"` za narekovaj). To je uporabno, ko želimo zapisati niz, ki vsebuje veliko posebnih znakov ali ko želimo ohraniti oblikovanje (npr. večvrstični nizi).

#### Sintaksa
Raw strings se začnejo in končajo z zaporedjem `r#` in `#`. Število `#` lahko povečamo, če je potrebno, da se izognemo konfliktu z vsebino niza.

Primeri:
```rust
let navaden_niz = "To je navaden niz z escape znaki: \nNova vrstica.";
let raw_string = r#"To je raw string brez escape znakov: \nNi nove vrstice."#;

let večvrstični_niz = r#"
To je večvrstični raw string.
Vsebuje več vrstic in ni potrebno uporabljati \n.
"#;

let niz_z_narekovaji = r#"Niz z "narekovaji" brez escape znakov."#;
```

#### Več `#` za kompleksne primere
Če niz vsebuje zaporedje `#`, lahko uporabimo več `#`, da se izognemo konfliktu:

```rust
let kompleksen_niz = r###"
Niz z "#" v vsebini in še več "#" okoli.
"###;
```

---

### Zaključek

- **Številski literali**: Rust podpira različne tipe števil in številske sisteme. Uporaba podčrtajev (`_`) omogoča bolj berljive številke.
- **Raw strings**: Uporabni so za zapis nizov brez uporabe escape znakov, še posebej pri večvrstičnih nizih ali nizih s posebnimi znaki.

