**Reference** in **borrowing** so ključni koncepti, ki omogočajo varno uporabo podatkov brez kopiranja ali prenosa lastništva podatkov memorije. Rust uporablja ownership sistem, ki zagotavlja, da se spomin upravlja varno in brez leakanja memorije.

#### 1. Reference

**Reference** so podobne pointerjem in omogočajo dostop do podatkov, ne da bi prevzeli njihov ownership.

- **Nemutabilne reference (`&T`)**: Omogočajo branje podatkov, vendar ne omogočajo spreminjanja.
- **Mutabilne reference (`&mut T`)**: Omogočajo tako branje kot spreminjanje podatkov.

Primer:

```rust
fn main() {
    let s1 = String::from("YO!");

    // Nemutabilne referenca
    let len = izracunaj_dolzino(&s1);

    println!("Dolžina '{s1}' je {len}.");
}

fn izracunaj_dolzino(s: &String) -> usize {
    s.len()
}
```

V tem primeru `&s1` ustvari nemutabilno referenco na `s1`. Funkcija `izracunaj_dolzino` lahko bere podatke, vendar jih ne more mutirati.

#### 2. Borrowing

**Borrowing** je koncept, kjer podatek posodite drugi funkciji ali delu kode, vendar lastništvo ostane pri originalnem lastniku. Taka izposoja zagotavlja, da podatki ne morejo mutirati na način, ki bi povzročil težave, kot so dvojno spremenljive reference ali spremembe med branjem.

- **Nemutabilen borrowing**: Več referenc je dovoljenih hkrati.
- **Mutabilen borrowing**: Le ena mutabilna referenca je dovoljena naenkrat.

Primer:

```rust
fn mutiraj(s: &mut String) {  
    s.push_str("truenjenir!");  
}  
fn main() {  
    let mut s = String::from("Yo, ");  
  
    mutiraj(&mut s);  
  
    println!("{s}");  
}
```

V tem primeru `&mut s` ustvari mutabilno referenco na `s`, ki dovoli mutacijo. Funkcija `spremeni` lahko mutira podatek, vendar samo ena mutabilna referenca je dovoljena naenkrat.

#### 3. Borrowing pravila

Rustova izposoja temelji na treh glavnih pravilih:

1. **Vsaka referenca mora biti veljavna**: Reference ne smejo kazati na neveljaven pomnilnik.
2. **Hkrati lahko imate eno spremenljivo referenco ali več nespremenljivih referenc**: To preprečuje podatkovne težave, kot so podatkovni prepleti.
3. **Reference ne smejo živeti dlje kot podatek, na katerega se sklicujejo**: To zagotavlja, da se ne uporablja neveljaven pomnilnik.

#### 4. Dangling reference

**Lifetime** je način zagotavljanja vedno veljavnih referenc. Lifetime je čas veljavnosti reference. Kompiler samodejno sklepa lifetime čase v večini primerov, včasih pa jih morate eksplicitno navesti.

Primer:

```rust
fn main() {  
    let s1 = String::from("Yo");  
    let s2 = String::from("truenjenir");  
  
    let najdaljsi = najdaljsi_string(&s1, &s2);  
  
    println!("Najdaljši string je {najdaljsi}");  
}  
  
fn najdaljsi_string<'a>(x: &'a str, y: &'a str) -> &'a str {  
    if x.len() > y.len() {  
        x  
    } else {  
        y  
    }
```

V tem primeru je `'a` lifetime čas, ki pove, da sta obe referenci `x` in `y` veljavni hkrati naenkrat isti cajt.
