**Closures** so anonimne funkcije, ki jih lahko shranimo v spremenljivko ali jih posredujemo kot argumente drugim funkcijam. So podobne funkcijam, vendar imajo nekaj dodatnih lastnosti:

1. **Lahko dostopajo do spremenljivk iz svojega območja (scope-a)**. To pomeni, da closure lahko "ujame" spremenljivke v območju, v katerem je bil ustvarjen.
2. **So fleksibilni glede tipov** – Rust samodejno sklepa tipe za parametre in vrnjene vrednosti, če jih ne navedemo eksplicitno.

### Sintaksa anonimnih funkcij (closures)
Closures v Rustu pišemo z uporabo dvojnih navpičnih črt `||`, ki vsebujejo parametre, če so. Telo closure pa je zaprto v zavitih oklepajih `{}`.

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
Ena od ključnih lastnosti anon. funkcij (closures) je, da lahko dostopajo do spremenljivk iz svojega območja. Na primer:

```rust
let x = 5;
let izpiši_x = || {
    println!("Vrednost x je: {x}");
};
izpiši_x();  // Izpiše: Vrednost x je: 5
```

V tem primeru closure `izpiši_x` "ujame" spremenljivko `x` iz svojega območja.

### Tipi anon. funkcij (closures)
Anon. funkcije (closures) so lahko treh vrst glede na to, kako uporabljajo spremenljivke iz okolja:

3. **`Fn`**: Closure, ki samo bere spremenljivke iz okolja.
4. **`FnMut`**: Closure, ki lahko spreminja spremenljivke iz okolja.
5. **`FnOnce`**: Closure, ki prevzame lastništvo nad spremenljivkami iz okolja (lahko se izvede samo enkrat).

Primer uporabe `FnMut`:

```rust
let mut števec = 0;
let povečaj_števec = || {
    števec += 1;
    println!("Števec: {števec}");
};
povečaj_števec();  // Števec: 1
povečaj_števec();  // Števec: 2
```

### Prenos lastništva z `move`
Če želimo, da closure prevzame lastništvo nad spremenljivkami iz okolja, uporabimo ključno besedo `move`:

```rust
let s = String::from("Pozdravljen, AI inženir!");
let closure = move || {
    println!("{s}");
};
closure();  // Izpiše: Pozdravljen, AI inženir!
// println!("{}", s);  // To ne bi delovalo, ker je `s` prestavljen v closure
```

### Zaključek
Anon. funkcije (closures) so močno orodje v Rustu, ki omogoča fleksibilnost in dostop do spremenljivk iz okolja. Uporabljajo se pogosto pri funkcijah, kot so `map`, `filter`, `fold` in podobno, kjer je potrebno posredovati logiko kot argument.

