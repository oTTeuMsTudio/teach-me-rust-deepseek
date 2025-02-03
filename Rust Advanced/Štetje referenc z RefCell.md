### Kaj je Reference Counting?

**Reference Counting** (štetje referenc) je tehnika, ki se uporablja za upravljanje lastništva (ownership) in življenjske dobe objektov v programskih jezikih. V Rustu je to doseženo z uporabo tipa **`Rc`** (Reference Counted). `Rc` omogoča, da ima več lastnikov na isti podatek, in samodejno osvobodi pomnilnik, ko zadnji lastnik izstopi iz obsega.

### Kaj je `RefCell`?

**`RefCell`** je struktura v Rustu, ki omogoča **interior mutability** (notranjo spremenljivost). To pomeni, da lahko spremenite vrednost, shranjeno v `RefCell`, tudi če je sama struktura nespremenljiva. `RefCell` uporablja dinamično preverjanje pravilnosti uporabe (borrow checking) namesto statičnega, kot to počne prevajalnik Rust.

### Kombinacija `Rc` in `RefCell`

Kombinacija `Rc` in `RefCell` je pogosto uporabljena, ko želimo imeti več lastnikov na isti podatek, ki ga želimo tudi spremeniti. `Rc` omogoča več lastništvo, `RefCell` pa omogoča spremenljivost.

### Primer uporabe `Rc<RefCell<T>>`

Poglejmo si primer, kako bi to izgledalo v praksi:

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    children: Vec<Rc<RefCell<Node>>>,
}

fn main() {
    // Ustvarimo vozlišče
    let leaf = Rc::new(RefCell::new(Node {
        value: 3,
        children: vec![],
    }));

    // Ustvarimo drugo vozlišče, ki ima `leaf` kot otroka
    let branch = Rc::new(RefCell::new(Node {
        value: 5,
        children: vec![Rc::clone(&leaf)],
    }));

    // Spremenimo vrednost v `leaf` prek `branch`
    branch.borrow_mut().children[0].borrow_mut().value = 10;

    // Izpišemo vrednosti
    println!("leaf: {:?}", leaf.borrow());
    println!("branch: {:?}", branch.borrow());
}
```

### Razlaga primera:

1. **Ustvarjanje vozlišč**:
   - `leaf` je vozlišče z vrednostjo `3` in brez otrok.
   - `branch` je vozlišče z vrednostjo `5` in ima `leaf` kot otroka.

2. **Spreminjanje vrednosti**:
   - `branch.borrow_mut()` nam da mutable referenco na `branch`.
   - `children[0].borrow_mut()` nam da mutable referenco na prvega otroka (`leaf`).
   - Nato spremenimo vrednost `leaf` na `10`.

3. **Izpis**:
   - Na koncu izpišemo vrednosti obeh vozlišč, da vidimo, da se je vrednost `leaf` res spremenila.

### Pomembne točke:

- **`Rc`**: Omogoča več lastništvo. Ko se zadnja referenca izgubi, se podatek samodejno osvobodi.
- **`RefCell`**: Omogoča spremenljivost, tudi če je struktura nespremenljiva. Uporablja dinamično preverjanje pravilnosti uporabe.
- **`borrow()` in `borrow_mut()`**: Metodi za pridobitev reference na podatek v `RefCell`. `borrow()` vrne nepremično referenco, `borrow_mut()` pa spremenljivo.

### Opozorila:

- **Ciklične reference**: `Rc` ne more samodejno odkriti cikličnih referenc, kar lahko povzroči uhajanje pomnilnika. V takih primerih je bolje uporabiti `Weak` (šibko referenco).
- **Panike**: Če poskušate pridobiti več kot eno mutable referenco na isti podatek, bo program paničiral.
