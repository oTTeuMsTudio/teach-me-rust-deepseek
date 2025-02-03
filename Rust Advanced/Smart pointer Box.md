### Kaj so Smart Pointerji?

**Smart Pointerji** so podatkovne strukture, ki se obnašajo kot običajni pointerji, vendar ponujajo dodatne funkcije, kot so **avtomatsko upravljanje pomnilnika** in **sledenje lastništvu**. 

### Kaj je `Box<T>`?

**`Box<T>`** je eden najpreprostejših smart pointerjev ki omogoča, da shranite podatke na **heap-u** namesto na **stack-u**. To je uporabno, ko:

1. Imate podatek, katerega velikost ni znana v času kompiliranja (npr. rekurzivni tipi).
2. Želite prenesti ownership velikega podatka, ne da bi ga kopirali.
3. Želite imeti pointer na podatek, ki ima fiksno velikost (npr. za prenos podatkov med različnimi deli programa).

### Kako uporabljati `Box<T>`?

#### Ustvarjanje `Box<T>`

Da ustvarite `Box`, uporabite funkcijo `Box::new`. Ta funkcija vzame vrednost in jo premakne na heap-allocated buffer.

```rust
fn main() {
    let stevilka = Box::new(42);  // Shrani število 42 na buffer
    println!("Številka v Box-u: {stevilka}");
}
```

V tem primeru je `številka` tipa `Box<i32>`. Ko `številka` pade iz obsega (out of scope), se pomnilnik samodejno sprosti.

#### Rekurzivni tipi z `Box<T>`

`Box` je še posebej uporaben pri rekurzivnih tipih, kjer velikost podatka ni znana vnaprej. Na primer, če želite ustvariti povezan seznam:

```rust
enum PovezanSeznam {
    Končnica,
    Vozlišče(i32, Box<PovezanSeznam>),
}

fn main() {
    let seznam = PovezanSeznam::Vozlišče(1, Box::new(PovezanSeznam::Vozlišče(2, Box::new(PovezanSeznam::Končnica))));
}
```

V tem primeru je `Box` uporabljen za shranjevanje pointerja na naslednji element seznama, kar omogoča rekurzivno strukturo.

#### Prenos lastništva

`Box` omogoča enostaven prenos lastništva, saj je `Box<T>` lastnik podatka. Ko `Box` pade iz obsega, se pomnilnik samodejno sprosti.

```rust
fn main() {
    let stevilka = Box::new(42);
    let druga_stevilka = stevilka;  // Lastništvo se prenese na `druga_številka`
    // println!("{}", številka);  // To bi povzročilo napako, ker `številka` ni več veljavna
    println!("{}", druga_stevilka);
}
```
