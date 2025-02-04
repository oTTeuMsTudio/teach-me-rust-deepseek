### Kaj je Reference Counting?

**Reference Counting** ali štetje referenc je tehnika, s katero kontroliramo pomniln. To doseženo z uporabo tipa `Rc<T>` (Reference Counted). `Rc<T>` omogoča, da imaš več ownerjev za eno vrednost ker samodejno sprosti pomnilnik, ko zadnji `owner` dropa iz obsega.

### Kaj pa Weak Referenca?

**Weak** referenca je posebna vrsta reference, ki ne povečuje števca referenc. To pomeni, da `Weak<T>` ne vpliva na `lifetime` podatkov. Če so vse `Rc<T>` reference izbrisane, bo vrednost izbrisana, tudi če še obstajajo `Weak<T>` reference.

### Kdaj uporabiti Weak Reference?

`Weak<T>` se uporablja, ko želiš imeti referenco na vrednost, vendar ne želiš, da ta referenca preprečuje izbris vrednosti. To je uporabno pri cikličnih referencah, kjer bi lahko `Rc<T>` povzročil leaking pomnilnika.

### Primer uporabe `Rc<T>` in `Weak<T>` v Rustu

Poglejmo si primer, kako uporabiti `Rc<T>` in `Weak<T>` v Rustu:

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    // Ustvarimo nov node z vrednostjo 42
    let leaf = Rc::new(Node {
        value: 42,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    // Ustvarimo še eno vozlišče, ki bo imelo `leaf` kot podrejenega
    let branch = Rc::new(Node {
        value: 10,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    // Nastavimo `parent` za `leaf` na `branch`
    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    // Izpišemo vrednosti
    println!("leaf value: {}", leaf.value);
    println!("branch value: {}", branch.value);

    // Dostopamo do nadrejeni (parent) iz `leaf`
    if let Some(parent) = leaf.parent.borrow().upgrade() {
        println!("leaf parent value: {}", parent.value);
    } else {
        println!("leaf has no parent");
    }
}
```

### Razlaga primera:

1. **Node struktura**: Vsako vozlišče (`Node`) ima vrednost (`value`), nadrejeni (`parent`), ki je `Weak<Node>`, in seznam podrejenih (`children`), ki so `Rc<Node>`.

2. **Ustvarjanje nodov**: Ustvarimo dva node-a, `leaf` in `branch`. `branch` ima `leaf` kot podrejenega.

3. **Nastavitev nadrejenega**: Nastavimo `parent` za `leaf` na `branch` z uporabo `Rc::downgrade`, ki ustvari `Weak<T>` referenco.

4. **Dostop do nadrejenega**: Z `upgrade()` poskušamo dobiti `Rc<T>` iz `Weak<T>`. Če nadrejeni še obstaja, ga lahko uporabimo.

### Zakaj uporabiti `Weak<T>`?

Če bi uporabili `Rc<T>` za `parent`, bi nastal ciklični odnos med `branch` in `leaf`, kar bi povzročilo leaking pomnilnika. Z `Weak<T>` se izognemo temu problemu, ker `Weak<T>` ne vpliva na števec referenc.

