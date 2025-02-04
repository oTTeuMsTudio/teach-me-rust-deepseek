Varno deljenje podatkov med thread-i rešujemo s večnitnostjo ali hkratnost, sočasnost (concurrency) `Mutex` in `Arc`.
### Kaj je `Mutex`?

`Mutex` je kratica za "Mutual Exclusion". To je sinhronizacijski primitiv, ki omogoča, da le en thread hkrati dostopa do deljenih podatkov. Ko thread želi dostopati do podatkov, mora najprej zakleniti (`lock`) `Mutex`. Če je `Mutex` že zaklenjen, bo nit čakala, dokler se ne odklene.

### Kaj je `Arc`?

`Arc` je kratica za "Atomic Reference Counting". To je smart pointer, ki omogoča večkratni ownership podatkov. `Arc` uporablja atomsko štetje referenc, da zagotovi, da se podatki ne bodo drop-al, dokler jih vsaj en thread še uporablja.

### Primer uporabe `Mutex` in `Arc`

Poglejmo si preprost primer, kjer več thread-ov posodablja skupen števec z uporabo `Mutex` in `Arc`.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // Ustvarimo števec, ki ga bomo delili med thread-i
    let counter = Arc::new(Mutex::new(0));

    // Vektor za hranjenje thread handl-ov
    let mut handles = vec![];

    for _ in 0..10 {
        // Kloniramo Arc, da ga lahko delimo med thread-i
        let counter = Arc::clone(&counter);

        // Ustvarimo nov thread
        let handle = thread::spawn(move || {
            // Zaklenemo Mutex, da dobimo dostop do podatkov
            let mut num = counter.lock().unwrap();

            // Posodobimo števec
            *num += 1;
        });

        // Dodamo thread handle-ov v vektor
        handles.push(handle);
    }

    // Počakamo, da se vsi thread-i zaključijo
    for handle in handles {
        handle.join().unwrap();
    }

    // Izpišemo končno vrednost števca
    println!("Končna vrednost števca: {}", *counter.lock().unwrap());
}
```

### Razlaga kode

1. **Ustvarjanje `Mutex` in `Arc`:**
   - `let counter = Arc::new(Mutex::new(0));` ustvari nov `Mutex`, ki vsebuje števec z začetno vrednostjo 0, in ga ovije v `Arc`, da ga lahko delimo med thread-i.

2. **Kloniranje `Arc`:**
   - `let counter = Arc::clone(&counter);` klonira `Arc`, kar poveča števec referenc. To omogoča, da vsak thread dobi lastno kopijo `Arc`, ki kaže na isti `Mutex`.

3. **Ustvarimo thread:**
   - `thread::spawn(move || { ... })` ustvari nov thread, ki izvaja `closure`. `move` prenese `ownership Arc` v thread.

4. **Zaklepanje `Mutex`:**
   - `let mut num = counter.lock().unwrap();` zaklene `Mutex`, da dobi izključni dostop do podatkov. `unwrap()` obdela morebitne napake.

5. **Posodabljanje števca:**
   - `*num += 1;` posodobi števec. Ker imamo izključni dostop, je to varno.

6. **Čakanje na thread:**
   - `handle.join().unwrap();` počaka, da se thread zaključi.

7. **Izpis končne vrednosti:**
   - `println!("Končna vrednost števca: {}", *counter.lock().unwrap());` izpiše končno vrednost števca.

### Zaključek

V tem primeru smo uporabili `Mutex` za zaščito deljenih podatkov in `Arc` za multi ownership. To je osnovni vzorec za hkratno večnitno programiranje, ki omogoča varno deljenje podatkov med thread-i.
