Tukaj bomo uporabili enum `Result` z generičnimi tipi podatkov. Enum `Result` se pogosto uporablja za obvladovanje napak. Definiran je takole:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

- `T` je generični tip, ki predstavlja uspešen rezultat.
- `E` je generični tip, ki predstavlja napako.

### Kako uporabljati `Result` z generičnimi tipi

Recimo, da imamo funkcijo, ki poskuša deliti dve števili. Če je deljenje z nič, želimo vrniti napako, sicer pa rezultat deljenja.

```rust
fn deli(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Deljenje z nič ni dovoljeno.".to_string())
    } else {
        Ok(a / b)
    }
}
```

V tem primeru:
- `T` je `f64`, ker je rezultat deljenja število s plavajočo vejico.
- `E` je `String`, ker napako predstavljamo kot niz.

### Uporaba `Result`-a v kodi

Ko kličemo funkcijo `deli`, moramo obravnavati obe možnosti: uspešen rezultat (`Ok`) in napako (`Err`).

```rust
fn main() {
    let rezultat = deli(10.0, 2.0);

    match rezultat {
        Ok(vrednost) => println!("Rezultat deljenja je: {vrednost}"),
        Err(napaka) => println!("Napaka: {napaka}"),
    }
}
```

V tem primeru:
- Če je `b` različen od nič, bo `deli` vrnil `Ok` z rezultatom deljenja.
- Če je `b` enak nič, bo `deli` vrnil `Err` z opisom napake.

### Dodatni primeri

#### Primer s celimi števili

```rust
fn deli_cela(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err("Deljenje z nič ni dovoljeno.".to_string())
    } else {
        Ok(a / b)
    }
}

fn main() {
    let rezultat = deli_cela(10, 0);

    match rezultat {
        Ok(vrednost) => println!("Rezultat deljenja je: {vrednost}"),
        Err(napaka) => println!("Napaka: {napaka}"),
    }
}
```

#### Primer z lastnimi tipi napak

Lahko definiramo tudi lastni tip napake:

```rust
#[derive(Debug)]
enum MojaNapaka {
    DeljenjeZNic,
}

fn deli_z_napako(a: f64, b: f64) -> Result<f64, MojaNapaka> {
    if b == 0.0 {
        Err(MojaNapaka::DeljenjeZNic)
    } else {
        Ok(a / b)
    }
}

fn main() {
    let rezultat = deli_z_napako(10.0, 0.0);

    match rezultat {
        Ok(vrednost) => println!("Rezultat deljenja je: {vrednost}"),
        Err(napaka) => println!("Napaka: {napaka:?}"),
    }
}
```

V tem primeru:
- `MojaNapaka` je naš lastni tip napake, ki ga uporabljamo namesto `String`.

### Zaključek

Enum `Result` z generičnimi tipi je zelo močno orodje za obvladovanje napak. Omogoča nam, da jasno ločimo med uspešnimi rezultati in napakami, ter da uporabljamo različne tipe za oboje. To naredi kodo bolj varno in berljivo.

