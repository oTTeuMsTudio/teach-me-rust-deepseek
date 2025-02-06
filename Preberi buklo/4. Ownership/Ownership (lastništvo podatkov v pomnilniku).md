Koncept **lastništva podatkov v pomnilniku** (ownership) je eden ključnih konceptov, ki zagotavlja varnost pri upravljanju s pomnilnikom brez potrebe po garbage collectorju.

---

### Kaj je ownership?

Vsaka vrednost ima **ownerja** (lastnika podatka), ki je spremenljivka, ki je odgovorna za sproščanje pomnilnika, ko vrednost ni več potrebna. Zato Rust uporablja ownership sistem, ki zagotovi pravilen management pomnilnika, ki prepreči podvajanje memorije in leaking.

Seveda! Razložil ti bom razliko med **skladom** (stack) in **kopico** (heap) v programiranju, s poudarkom na tem, kako se to uporablja v Rustu. Oba koncepta sta ključna za razumevanje, kako programi upravljajo s pomnilnikom.

---

# Stack in heap

**Stack** je del pomnilnika, kjer se shranjujejo podatki, katerih velikost je znana v času kompilacije. Deluje po principu **LIFO** (Last In, First Out), kar pomeni, da se zadnji dodani podatek prvi odstrani.

#### Značilnosti pomnilnika stack:
- **Hitrost**: Dostop je zelo hiter, ker je organiziran kot stack podatkov.
- **Avtomatik management**: Ko spremenljivka izstopi iz območja funkcije, se njeni podatki samodejno odstranijo iz njenega območja.
- **Fiksna velikost**: Podatki morajo imeti znano velikost pred kompilacijo.

Primer v Rustu:
```rust
let x = 5; // `x` je shranjen na skladu, ker je njegova velikost znana (32-bitno celo število)
let y = 10; // `y` je prav tako na skladu
```

---

### Heap

**Heap** je del pomnilnika, kjer se shranjujejo podatki, katerih velikost ni znana pred kompiliranjem ampak dinamično med uporabo programov, je bolj fleksibilen in zahteva ročno upravljanje pomnilnika (ali pa se zanaša na sisteme, kot je ownership sistem).

#### Značilnosti pomnilnika heap:
- **Dinamična velikost**: Podatki tukaj rastejo in se skrčijo med izvajanjem programa.
- **Počasnejši dostop**: Dostop je počasnejši, ker mora program najprej poiskati prostor v pomnilniku.
- **Ročno upravljanje**: V jezikih, kot sta C ali C++, mora programer ročno sprostiti pomnilnik. Tukaj to avtomatsko menegira ownership sistem.

Primer:
```rust
let s = String::from("yo truenjenir!"); // `s` je shranjen na kopici, ker je njegova velikost lahko spremenljiva
```

---

### Primerjava stacka in heapa

| Značilnost            | Stack                     | Heap                              |
| --------------------- | ------------------------- | --------------------------------- |
| **Hitrost**           | Zelo hiter                | Počasnejši                        |
| **Velikost podatkov** | Znana pred kompiliranjem  | Dinamična, med uporabo programa   |
| **Upravljanje**       | Avtomatsko                | Ročno ali preko sistema ownership |
| **Uporaba**           | Za majhne, fiksne podatke | Za velike in dinamične podatke    |

---

### Kako Rust uporablja stack in heap?

Podatki, kot so celoštevilske vrednosti (`i32`, `u64`), logične vrednosti (`bool`) in fiksni stringi (`&str`) se shranjujejo na **steku**, ker imajo znano velikost.

Podatki, kot so dinamični string (`String`), vektorji (`Vec<T>`) ali structs z dinamično velikostjo, pa se shranjujejo na **heapu**. Rust uporablja ownership sistem in sistem borrow, da zagotovi varno upravljanje s heap-alociranim bufferjem brez leakanja.

Primer:
```rust
let stack_data = 42; // Shranjeno na skladu
let heap_data = String::from("Rust je kul!"); // Shranjeno na kopici
```

---

### Glavna pravila lastništva podatkov memorije

1. **Vsaka vrednost ima samo enega ownerja.**
   - Vsaka vrednost ima lahko samo eno spremenljivko, ki je njen owner.
   - Ko lastnik drop-a iz področja funkcije, se vrednost avtomatično sprosti.

   Primer:
   ```rust
   let s = String::from("yo truenjenir!"); // `s` je owner vrednosti
   ```

2. **Vrednost ima lahko samo enega lastnika naenkrat.**
   - Če vrednost prenesemo na drugo spremenljivko, prvotni lastnik ne more več uporabljati te vrednosti.

   Primer:
   ```rust
   let s1 = String::from("YO!");
   let s2 = s1; // `s1` se prenese na `s2`, `s1` ni več veljaven
   // println!("{}", s1); // To bi povzročilo napako, ker `s1` ni več lastnik
   ```

3. **Ko owner izstopi iz obsega, se vrednost sprosti.**
   - Rust samodejno sprosti pomnilnik, ko spremenljivka izstopi iz obsega (npr. na koncu funkcije).

   Primer:
   ```rust
   {
       let s = String::from("Yo!");
       // `s` je v uporabi
   } // `s` izstopi iz obsega, pomnilnik se sprosti
   ```

---

### Prenos ali interakcija podatkov in spremenljivk z Move

Ko vrednost prenesemo na drugo spremenljivko, se lastništvo **prenese** (move). Prvotna spremenljivka ne more več uporabljati te vrednosti.

Primer:
```rust
let s1 = String::from("Rust");
let s2 = s1; // Lastništvo se prenese iz `s1` na `s2`
// println!("{}", s1); // Napaka: `s1` ni več veljaven
```

---

### Kloniranje (Clone)

Če želimo ustvariti kopijo vrednosti in ohraniti prvotno lastništvo, lahko uporabimo metodo `.clone()`.

Primer:
```rust
let s1 = String::from("Rust");
let s2 = s1.clone(); // `s1` in `s2` sta neodvisni kopiji
println!("{} {}", s1, s2); // Deluje brez napake
```

---

### Reference in borrowing

Vrednost si tukaj **izposodimo** (borrow) brez prevzema lastništva. To storimo z referencami (`&`).

Primer:
```rust
let s1 = String::from("Rust");
let s2 = &s1; // `s2` je referenca na `s1`, lastništvo ostaja pri `s1`
println!("{} {}", s1, s2); // Deluje brez napake
```

Reference so lahko:
- **Nespremenljive** (`&T`): Omogočajo branje, ne pa tudi spreminjanja.
- **Spremenljive** (`&mut T`): Omogočajo spreminjanje vrednosti.

