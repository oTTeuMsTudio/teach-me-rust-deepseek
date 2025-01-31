**closures** so anonimne funkcije, ki jih lahko shranimo v spremenljivko ali jih posredujemo kot argumente drugim funkcijam. So podobne funkcijam, vendar imajo nekaj dodatnih lastnosti:

1. **Lahko dostopajo do spremenljivk iz svojega okolja** (scope). To pomeni, da lahko closure "ujame" spremenljivke iz okolja, v katerem je bila ustvarjena.
2. **So fleksibilne glede tipov** – Rust samodejno sklepa tipe za parametre in vrnjene vrednosti, če jih ne navedemo eksplicitno.

### Sintaksa closures
Closures v Rustu pišemo z uporabo dvojnih navpičnih črt `||`, ki vsebujejo parametre, če jih je treba. Telo closure pa je zaprto v zavitih oklepajih `{}`.

```rust
let closure = || {
    println!("To je closure!");
};
```

Če ima closure parametre, jih navedemo med `||`:

```rust
let seštej = |a, b| a + b;
println!("Rezultat: {}", seštej(2, 3));  // Izpiše: Rezultat: 5
```

### Ujemanje spremenljivk iz okolja
Ena od ključnih lastnosti closures je, da lahko dostopajo do spremenljivk iz svojega okolja. Na primer:

```rust
let x = 5;
let izpiši_x = || {
    println!("Vrednost x je: {}", x);
};
izpiši_x();  // Izpiše: Vrednost x je: 5
```

V tem primeru closure `izpiši_x` "ujame" spremenljivko `x` iz svojega okolja.

### Tipi closures
Closures so lahko treh vrst glede na to, kako uporabljajo spremenljivke iz okolja:

1. **`Fn`**: Closure, ki samo bere spremenljivke iz okolja.
2. **`FnMut`**: Closure, ki lahko spreminja spremenljivke iz okolja.
3. **`FnOnce`**: Closure, ki prevzame lastništvo nad spremenljivkami iz okolja (lahko se izvede samo enkrat).

Primer uporabe `FnMut`:

```rust
let mut števec = 0;
let povečaj_števec = || {
    števec += 1;
    println!("Števec: {}", števec);
};
povečaj_števec();  // Števec: 1
povečaj_števec();  // Števec: 2
```

### Prenos lastništva z `move`
Če želimo, da closure prevzame lastništvo nad spremenljivkami iz okolja, uporabimo ključno besedo `move`:

```rust
let s = String::from("Pozdravljen, svet!");
let closure = move || {
    println!("{}", s);
};
closure();  // Izpiše: Pozdravljen, svet!
// println!("{}", s);  // To ne bi delovalo, ker je `s` prestavljen v closure
```

### Zaključek
Closures so močno orodje v Rustu, ki omogoča fleksibilnost in dostop do spremenljivk iz okolja. Uporabljajo se pogosto pri funkcijah, kot so `map`, `filter`, `fold` in podobno, kjer je potrebno posredovati logiko kot argument.

Če imaš kakršnakoli vprašanja ali potrebuješ dodatne razlage, le vprašaj! 😊
