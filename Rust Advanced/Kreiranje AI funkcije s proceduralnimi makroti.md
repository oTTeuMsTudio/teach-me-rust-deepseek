### Kaj bomo zgradili?

Zgradili bomo preprosto AI funkcijo, ki bo uporabljala proceduralne makrote za generiranje kode, ki izvaja osnovne matematične operacije. To bo osnova za bolj zapletene AI algoritme v prihodnosti.

### Korak 1: Nastavitev projekta

Najprej ustvarimo nov Rust projekt:

```bash
cargo new ai_function
cd ai_function
```

### Korak 2: Dodajanje potrebnih odvisnosti

V `Cargo.toml` dodajte naslednje dependente:

```toml
[lib]
proc-macro = true

[dependencies]
syn = "1.0"
quote = "1.0"
```

- `syn`: Knjižnica za razčlenjevanje Rust kode.
- `quote`: Knjižnica za generiranje Rust kode.

### Korak 3: Ustvarjanje proceduralnega makra

V `src/lib.rs` ustvarimo naš proceduralni makro. Ta makro bo generiral funkcijo, ki izvaja matematične operacije.

```rust
extern crate proc_macro;

use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, ItemFn};

#[proc_macro]
pub fn ai_function(input: TokenStream) -> TokenStream {
    // Razčleni vhodno funkcijo
    let input_fn = parse_macro_input!(input as ItemFn);

    // Ime funkcije
    let fn_name = &input_fn.sig.ident;

    // Generiraj novo funkcijo
    let expanded = quote! {
        fn #fn_name(a: i32, b: i32) -> i32 {
            a + b // Preprosta seštevalna funkcija
        }
    };

    // Vrnite generirano kodo kot TokenStream
    TokenStream::from(expanded)
}
```

### Korak 4: Uporaba makra v glavnem programu

V `src/main.rs` uporabimo naš makro za generiranje AI funkcije:

```rust
use ai_function::ai_function;

// Uporaba makra za generiranje funkcije
ai_function!(add);

fn main() {
    let result = add(3, 4);
    println!("Rezultat: {}", result); // Izpis: Rezultat: 7
}
```

### Korak 5: Zagon programa

Zdaj lahko zaženete program z ukazom:

```bash
cargo run
```

Program bi moral izpisati `Rezultat: 7`, kar pomeni, da je naša AI funkcija uspešno generirana in izvedena.

### Razlaga

- **Proceduralni makri**: V tem primeru smo uporabili proceduralni makro `ai_function`, ki generira novo funkcijo `add` v času prevajanja. Ta funkcija izvaja preprosto seštevanje dveh števil.
- **`syn` in `quote`**: Te knjižnice omogočajo razčlenjevanje in generiranje Rust kode. `syn` razčleni vhodno kodo, `quote` pa generira novo kodo.
- **Uporaba makra**: V `main.rs` smo uporabili makro `ai_function!`, da smo generirali funkcijo `add`, ki jo nato uporabili v glavnem programu.

### Nadgradnja

To je le osnova. Lahko nadgradite to AI funkcijo, da izvaja bolj zapletene operacije, kot so množenje, deljenje ali celo izvajanje nevronskih mrež. Proceduralni makri so zelo močno orodje, ki vam omogoča, da avtomatizirate generiranje kode in ustvarite bolj fleksibilne in močne programe.

### Zaključek

V tej vadnici smo se naučili, kako uporabljati proceduralne makre v Rustu za generiranje kode v času prevajanja. Zgradili smo preprosto AI funkcijo, ki izvaja matematične operacije. To je le začetek, proceduralni makri pa so zelo močno orodje za avtomatizacijo in generiranje kode v Rustu.
