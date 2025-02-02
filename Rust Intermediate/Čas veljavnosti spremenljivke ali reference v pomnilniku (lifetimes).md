V nizkonivojskem jeziku Rust poznamo koncept **časa veljavnosti podatkov v pomnilniku** (lifetimes), kar je del sistema lastništva, ki pomaga preprečiti napake z dostopom do neveljavnih podatkov.

### Kaj so lifetimes?

Lifetimes se nanašajo na to, kako dolgo je neka spremenljivka ali referenca veljavna. Rust uporablja lifetimes, da zagotovi, da reference vedno kažejo na veljavne podatke. To je pomembno, ker preprečuje napake, kot so "dangling pointers" (reference na neveljavne podatke).

### Sintaksa `lifetimes`-ov

Lifetimes se označujejo z apostrofom (`'`), ki mu sledi identifikator, na primer `'a`, `'b`, itd. Te oznake se uporabljajo za povezovanje `lifetime`-ov različnih referenc.

Primer:

```rust
fn najdaljsi<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

V tem primeru:

- `'a` je veljavnost reference, ki se nanaša na obe vhodni referenci (`x` in `y`) in na izhodno referenco.
- Funkcija `najdaljsi` vrne referenco na daljši niz, ki ima isto veljavno dobo kot `input` parametri.

### Zakaj so `lifetimes` dobe pomembne?

Rust-ova prevajalnik (compiler) uporablja `lifetime` dobe, da zagotovi, da reference ne bodo preživele podatke, na katere se sklicujejo. To pomeni, da Rust preprečuje napake, kot so:

- **Dangling references**: Ko referenca kaže na podatek, ki je bil že sproščen iz pomnilnika.
- **Use-after-free**: Ko poskušamo uporabiti podatek, ki je bil že sproščen.

### Primer brez `lifetimes` dobe

Če bi poskušali napisati funkcijo brez določenih `lifetimes` dobe, bi Rust-ov prevajalnik vrnil napako:

```rust
fn najdaljsi(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Napaka bi bila podobna tej:

```
error[E0106]: missing lifetime specifier
 --> src/main.rs:1:33
  |
1 | fn najdaljsi(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
```

Rust zahteva, da eksplicitno določimo `lifetimes` dobe, da se izognemo morebitnim napakam.

### `lifetimes` dobe v strukturah

`Lifetimes` dobe se uporabljajo tudi v strukturah (`structs`), ki vsebujejo reference. Na primer:

```rust
struct Oseba<'a> {
    ime: &'a str,
    starost: u32,
}

fn main() {
    let ime = String::from("Janez");
    let oseba = Oseba { ime: &ime, starost: 30 };
    println!("{} je star {} let.", oseba.ime, oseba.starost);
}
```

V tem primeru:

- `Oseba` ima referenco na `&str` (`&ime`), zato moramo določiti `lifetime 'a`.
- To zagotovi, da `Oseba` ne bo preživela podatka, na katerega se sklicuje.

### Sklep

Lifetimes dobe so orodje v nizkonivojskem jeziku Rust, ki pomagajo zagotoviti varnost pomnilnika. 
