Proceduralni makroti kot napredna funkcionalnost v nizkonivojskem jeziku Rust omogoča metaprogramiranje.  Proceduralni makroti so bolj fleksibilni in močnejši od deklarativnih makrotov (ki jih definiraš z `macro_rules!`).

### Funkcijski makroti

Poleg **derive** makrov in **attribute** makrov so funkcijski makroti podobni klasičnim funkcijam.

### Kako delujejo?

Funkcijski makroti se definirajo s pomočjo posebnih knjižnic, kot je `proc_macro`. Ti makroti sprejmejo nek `input` (običajno tok tokenov), obdelajo ta `input` in vrnejo novo kodo, ki jo prevajalec nato uporabi.

### Primer: Preprost funkcijski makro

Poglejmo si preprost primer, ki generira funkcijo, ki izpiše "Pozdravljen, svet!".

```rust
extern crate proc_macro;
use proc_macro::TokenStream;
use quote::quote;

#[proc_macro]
pub fn pozdravi(_input: TokenStream) -> TokenStream {
    let razširjena_koda = quote! {
        fn pozdravi() {
            println!("Pozdravljen, svet!");
        }
    };
    razširjena_koda.into()
}
```

### Razlaga:

1. **`extern crate proc_macro;`**: Uvozimo knjižnico `proc_macro`, ki nam omogoča delo s proceduralnimi makri.

2. **`use proc_macro::TokenStream;`**: `TokenStream` je osnovni tip, ki predstavlja tok tokenov, ki jih makro obdela.

3. **`#[proc_macro]`**: Ta atribut označuje, da je funkcija `pozdravi` proceduralna makra.

4. **`quote!`**: Makro `quote!` iz knjižnice `quote` nam omogoča enostavno generiranje kode. V tem primeru generiramo funkcijo `pozdravi`, ki izpiše "Pozdravljen, svet!".

5. **`razširjena_koda.into()`**: Pretvorimo generirano kodo v `TokenStream`, ki ga Rust prevajalec lahko uporabi.

### Uporaba makra

Ko imamo makro definiran, ga lahko uporabimo v drugem delu kode:

```rust
use moj_makro::pozdravi;

pozdravi!();

fn main() {
    pozdravi();
}
```

Ko prevajalec prevede to kodo, bo makro `pozdravi!` generiral funkcijo `pozdravi()`, ki jo lahko kličemo v `main`.

### Zakaj so funkcijski makroti uporabni?

Uporabljamo jih, ker omogočajo:

- **Generiranje kode**: Lahko generiraš ponavljajočo se kodo, kar zmanjša možnost napak.
- **Dinamično programiranje**: Lahko ustvariš kodo, ki je odvisna od vnosa v času prevajanja.
- **Razširljivost**: Omogočajo razširjanje jezika Rust z novimi funkcionalnostmi.

### Omejitve

- **Kompleksnost**: Proceduralne makre so bolj kompleksne za pisanje in razumevanje kot deklarativne makre.
- **Kompilirni čas**: Ker se makri izvajajo v času prevajanja, lahko povečajo čas prevajanja.
