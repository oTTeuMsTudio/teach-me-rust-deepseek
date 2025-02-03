Atributi so metapodatki, ki jih lahko dodate h različnim elementom v vaši kodi, kot so funkcije, strukture, moduli itd. Ti atributi lahko vplivajo na to, kako se koda kompilira ali kako se obnaša.

### Osnovna Sintaksa

Atributi v Rustu so zapisani z `#` in so običajno zaprti v oglate oklepaje `[]`. Najpogostejši atribut je `#[derive]`, ki se uporablja za samodejno implementacijo določenih lastnosti (traits) za vaše strukture ali enumeracije.

Primer:

```rust
#[derive(Debug)]
struct Oseba {
    ime: String,
    starost: u32,
}

fn main() {
    let oseba = Oseba {
        ime: String::from("Janez"),
        starost: 30,
    };

    println!("{:?}", oseba);
}
```

V tem primeru smo uporabili atribut `#[derive(Debug)]`, ki samodejno implementira lastnost `Debug` za strukturo `Oseba`. To nam omogoča, da uporabimo makro `println!` z `{:?}` za izpis strukture.

### Pogosto Uporabljeni Atributi

1. **`#[derive]`**: Samodejno implementira lastnosti kot so `Debug`, `Clone`, `Copy`, `PartialEq`, itd.

2. **`#[allow]` in `#[deny]`**: Omogočata ali prepovedata določena opozorila (warnings) v vaši kodi.

   ```rust
   #[allow(dead_code)]
   fn neuporabljena_funkcija() {
       println!("Ta funkcija se ne uporablja.");
   }
   ```

3. **`#[cfg]`**: Omogoča pogojno prevajanje kode glede na določene pogoje, kot je na primer operacijski sistem.

   ```rust
   #[cfg(target_os = "linux")]
   fn samo_za_linux() {
       println!("Ta koda se prevaja samo za Linux.");
   }
   ```

4. **`#[test]`**: Označuje funkcijo kot testno funkcijo, ki se izvede samo, ko zaženete teste.

   ```rust
   #[test]
   fn test_seštevanja() {
       assert_eq!(2 + 2, 4);
   }
   ```

5. **`#[macro_use]`**: Omogoča uporabo makrov iz drugega modula.

   ```rust
   #[macro_use]
   mod moji_makri {
       macro_rules! pozdravi {
           () => {
               println!("Pozdravljen, svet!");
           };
       }
   }

   fn main() {
       pozdravi!();
   }
   ```

### Ustvarjanje Lastnih Atributov

V Rustu lahko ustvarite tudi svoje lastne atribute z uporabo **proceduralnih makrov**. To je napredna tema, ki zahteva ustvarjanje posebne knjižnice (crate) za proceduralne makre.

Primer:

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_attribute]
pub fn moj_atribut(_attr: TokenStream, item: TokenStream) -> TokenStream {
    let input = syn::parse_macro_input!(item as syn::ItemFn);
    let fn_name = &input.sig.ident;

    let output = quote! {
        fn #fn_name() {
            println!("Ta funkcija je bila spremenjena z atributom!");
        }
    };

    TokenStream::from(output)
}
```

V tem primeru smo ustvarili lasten atribut `moj_atribut`, ki spremeni funkcijo, na katero je uporabljen.

