Rust je nizkonivojski sistemski programski jezik, ki ponuja različne vrste makrov za metaprogramiranje. Proceduralni makri so ena izmed teh vrst.

### Kaj so Proceduralni Makri?

Proceduralni makri so funkcije, ki sprejmejo nek `input` (običajno Rust kodo) in vrnejo nek `output` (spet Rust kodo). Ti makroti se izvajajo v času kompiliranja in se uporabljajo za generiranje ali spreminjanje kode. Poznamo tri glavne vrste proceduralnih makrov:

1. **Funkcijski makri**: Makri, ki se uporabljajo kot funkcije.
2. **Atributni makri**: Makri, ki se uporabljajo kot atributi nad strukturami, enumeracijami, funkcijami itd.
3. **Derive makri**: Makri, ki se uporabljajo za avtomatsko implementacijo lastnosti (traits) za strukture ali enumeracije.

### Derive Makroti

Derive makri so posebna vrsta proceduralnih makrov, ki omogočajo avtomatsko implementacijo lastnosti (traits) za vaše strukture ali enumeracije. Na primer, če imate strukturo in želite, da implementira lastnost `Debug`, lahko uporabite `#[derive(Debug)]`, da prevajalnik samodejno generira ustrezno implementacijo.

#### Primer

```rust
#[derive(Debug)]
struct Oseba {
    ime: String,
    starost: u8,
}

fn main() {
    let oseba = Oseba {
        ime: String::from("Janez"),
        starost: 30,
    };

    println!("{:?}", oseba);
}
```

V tem primeru smo uporabili `#[derive(Debug)]`, da smo avtomatsko implementirali lastnost `Debug` za strukturo `Oseba`. To nam omogoča, da izpišemo strukturo z uporabo `println!` makra.

### Kako Delujejo Derive Makri?

Ko uporabite `#[derive(Trait)]`, Rust prevajalnik kliče proceduralni makro, ki generira kodo za implementacijo te lastnosti. Ta makro prejme strukturo ali enumeracijo kot `input` in vrne ustrezno implementacijo lastnosti.

#### Primer Implementacije

Če bi želeli implementirati lastnost `Debug` ročno, bi to naredili takole:

```rust
use std::fmt;

struct Oseba {
    ime: String,
    starost: u8,
}

impl fmt::Debug for Oseba {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "Oseba {{ ime: {}, starost: {} }}", self.ime, self.starost)
    }
}

fn main() {
    let oseba = Oseba {
        ime: String::from("Janez"),
        starost: 30,
    };

    println!("{:?}", oseba);
}
```

Z uporabo `#[derive(Debug)]` pa nam ni treba pisati te kode ročno, saj jo prevajalnik samodejno generira.

### Ustvarjanje Lastnih Derive Makrov

Če želite ustvariti svoj lasten derive makro, morate uporabiti knjižnico `syn` za razčlenjevanje vhodne kode in `quote` za generiranje izhodne kode. Tukaj je osnovni primer:

```rust
extern crate proc_macro;
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(MojTrait)]
pub fn moj_trait_derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    let ime = &ast.ident;

    let gen = quote! {
        impl MojTrait for #ime {
            fn moja_metoda(&self) {
                println!("To je moja metoda za struct {}", stringify!(#ime));
            }
        }
    };

    gen.into()
}
```

V tem primeru smo ustvarili derive makro `MojTrait`, ki implementira metodo `moja_metoda` za strukturo, na katero ga uporabimo.

### Zaključek

Proceduralni makri, zlasti derive makroti omogočajo avtomatizacijo. Z njimi lahko znatno zmanjšate količino ponavljajoče se kode in izboljšate berljivost ter vzdržljivost vaših programov.
