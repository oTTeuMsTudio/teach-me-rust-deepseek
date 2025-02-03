Deklarativni makro omogoča generiranje kode v času kompilacije. Ti makroti so podobni funkcijam, vendar se izvajajo med kompilacijo in lahko generirajo kodo, ki se nato vstavi v naš program.

### Osnove deklarativnih makrov

Deklarativni makri se definirajo z uporabo `macro_rules!`. Ta makro omogoča definiranje ukazov, ki se ujemajo z `input` podatki, in kode, ki se generira, ko se ukaz ujema.

#### Primer preprostega makra

Recimo, da želimo ustvariti makro, ki pozdravi uporabnika:

```rust
macro_rules! pozdravi {
    () => {
        println!("Pozdravljen, svet!");
    };
    ($ime:expr) => {
        println!("Pozdravljen, {}!", $ime);
    };
}

fn main() {
    pozdravi!(); // Izpiše: "Pozdravljen, svet!"
    pozdravi!("Ana"); // Izpiše: "Pozdravljen, Ana!"
}
```

V tem primeru:

- `macro_rules! pozdravi` definira makro z imenom `pozdravi`.
- Prvi ukaz `()` ujema klic makra brez argumentov in izpiše "Pozdravljen, svet!".
- Drugi ukaz `($ime:expr)` ujema klic makra z enim argumentom (parametrom) in izpiše pozdrav s tem imenom.

### Sestavni deli makrov

1. **Ukaz (Pattern)**: To je del, ki se ujema z `input` podatki. Ukaz lahko vključuje literale, identifikatorje, izraze itd.

2. **Telo (Body)**: To je koda, ki se generira, ko se ukaz ujema. Telo lahko vključuje katero koli veljavno Rust kodo.

3. **Meta spremenljivke (Metavariables)**: To so spremenljivke, ki se uporabljajo v ukazih za zajemanje `input` podatkov. Na primer, `$ime:expr` zajame izraz in ga shrani v spremenljivko `$ime`.

### Več o ukazih

Ukazi v makrih so zelo fleksibilni. Tukaj je nekaj pogostih vzorcev:

- `expr`: Ujema se z izrazom (npr. `5 + 3`, `x`, `foo()`).
- `ident`: Ujema se z identifikatorjem (npr. ime spremenljivke ali funkcije).
- `ty`: Ujema se z tipom (npr. `i32`, `String`).
- `pat`: Ujema se z ukazom (npr. `Some(x)`, `_`).

#### Primer makra z več ukazi

```rust
macro_rules! izracunaj {
    ($a:expr, $b:expr) => {
        $a + $b
    };
    ($a:expr, $b:expr, $c:expr) => {
        $a + $b + $c
    };
}

fn main() {
    let vsota1 = izracunaj!(1, 2); // 3
    let vsota2 = izracunaj!(1, 2, 3); // 6
    println!("Vsota1: {}, Vsota2: {}", vsota1, vsota2);
}
```

V tem primeru makro `izracunaj!` sprejme dva ali tri argumente in vrne njihovo vsoto.

### Ponavljanje v makrih

Makri podpirajo tudi ponavljanje, kar omogoča generiranje kode za spremenljivo število argumentov.

#### Primer makra s ponavljanjem

```rust
macro_rules! seznam {
    ($($element:expr),*) => {
        {
            let mut seznam = Vec::new();
            $(seznam.push($element);)*
            seznam
        }
    };
}

fn main() {
    let s = seznam![1, 2, 3, 4];
    println!("{:?}", s); // Izpiše: [1, 2, 3, 4]
}
```

V tem primeru:

- `$($element:expr),*` ujema seznam izrazov, ločenih z vejico.
- `$(seznam.push($element);)*` ponavlja kodo za vsak element v seznamu.

### Zaključek

Deklarativni makri so orodje za generiranje kode v času kompiliranja. Z uporabo `macro_rules!` lahko ustvarite makre, ki sprejemajo različne ukaze in generirajo kodo, ki jo potrebujete. To lahko znatno poenostavi vašo kodo in jo naredi bolj fleksibilno.
