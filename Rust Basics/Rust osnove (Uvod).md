# Uvod v slovenščini
Rust je varen programski jezik, kjer je poudarek na hitrosti, paralelnosti in varnosti pri upravljanju s spominom.

### 1. Namestitev Rust-a
Najprej morate namestiti Rust. To lahko storite s pomočjo orodja `rustup`, ki vam omogoča enostavno upravljanje različic Rusta.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Po namestitvi lahko preverite, ali je Rust pravilno nameščen:

```bash
rustc --version
```

### 2. Prvi program v Rust-u
Ustvarimo nov projekt:

```bash
cargo new moj_prvi_projekt
cd moj_prvi_projekt
```

V mapi `src` boste našli datoteko `main.rs`. To je vaš glavni program. Odprite jo in uredite:

```rust
fn main() {
    println!("Pozdravljen, svet!");
}
```

Za zagon programa uporabite:

```bash
cargo run
```

### 3. Osnove Rust-a

#### Spremenljivke
V Rustu so spremenljivke privzeto nespremenljive (`immutable`). Če želite spremenljivo spremenljivko, uporabite ključno besedo `mut`.

```rust
fn main() {
    let x = 5; // nespremenljiva spremenljivka
    let mut y = 10; // spremenljiva spremenljivka
    y = y + 1;
    println!("x = {}, y = {}", x, y);
}
```

#### Podatkovni tipi
Rust ima močno statično tipizacijo. Nekaj osnovnih tipov:

- Celoštevilski tipi: `i32`, `u32`, `i64`, `u64`, itd.
- Plavajoča vejica: `f32`, `f64`
- Logični tip: `bool`
- Znaki: `char`
- Nizi: `String`, `&str`

```rust
fn main() {
    let celo_stevilo: i32 = 42;
    let plavajoca_vejica: f64 = 3.14;
    let logicna_vrednost: bool = true;
    let znak: char = 'R';
    let niz: &str = "Zdravo, Rust!";

    println!("{}, {}, {}, {}, {}", celo_stevilo, plavajoca_vejica, logicna_vrednost, znak, niz);
}
```

#### Funkcije
Funkcije se definirajo s ključno besedo `fn`.

```rust
fn pozdrav(ime: &str) {
    println!("Pozdravljen, {}!", ime);
}

fn main() {
    pozdrav("g. Inženir");
}
```

#### Kontrolne strukture
Rust podpira običajne kontrolne strukture, kot so `if`, `else`, `loop`, `while`, in `for`.

```rust
fn main() {
    let številka = 7;

    if številka < 5 {
        println!("Pogoj je bil izpolnjen");
    } else {
        println!("Pogoj ni bil izpolnjen");
    }

    let rezultat = if številka > 5 { "večje" } else { "manjše" };
    println!("Rezultat je {}", rezultat);

    // Zanka
    let mut števec = 0;
    loop {
        števec += 1;
        if števec == 3 {
            break;
        } println!("števec loop = {}", števec);
    }

    // While zanka
    while števec < 5 {
        števec += 1;
    } println!("števec while = {}", števec);

    // For zanka
    for i in 0..5 {
        println!("i = {}", i);
    }
}
```

#### Lastništvo in izposoja
Ena najpomembnejših lastnosti Rust-a je sistem lastništva (`ownership`), ki zagotavlja varnost pri upravljanju s spominom brez uporabe garbage collector-ja.

- Vsaka vrednost ima lastnika.
- Vrednost ima lahko samo enega lastnika naenkrat.
- Ko lastništvo vrednosti zapusti območje funkcije (oglate oklepaje), se vrednost izbriše iz spomina.

```rust
fn main() {
    let s1 = String::from("Pozdravljen");
    let s2 = s1; // s1 se premakne v s2 in ni več veljaven

    // println!("{}", s1); // To bi povzročilo napako
    println!("{}", s2);
}
```

Izposoja (`borrowing`) omogoča, da se vrednost posodi brez prevzema lastništva.

```rust
fn main() {
    let s1 = String::from("Pozdravljen");
    let dolzina = izracunaj_dolzino(&s1); // Posojamo s1 oz. sposojamo si od s1
    println!("Dolžina '{}' je {}.", s1, dolzina);
}

fn izracunaj_dolzino(s: &String) -> usize {
    s.len()
}
```

### 4. Dodatni viri
- [Uradna dokumentacija Rusta](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)

To so samo osnove Rusta. Jezik ima še veliko več funkcij, kot so strukture, enumeracije, vzorci, lastnosti (`traits`), življenjske dobe (`lifetimes`), in še veliko več. S prakso in študijem boste postali bolj spretni v Rustu. Veliko sreče pri učenju! 🚀
