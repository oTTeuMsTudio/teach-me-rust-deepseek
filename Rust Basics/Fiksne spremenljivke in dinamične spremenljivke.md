Seveda! Naučimo se o **fiksno velikih spremenljivkah** in **dinamično velikih spremenljivkah** v Rustu. Razlaga bo v slovenščini, s primeri za boljše razumevanje.

---

### **Fiksno velike spremenljivke**
Fiksno velike spremenljivke imajo velikost, ki je znana že v času prevajanja (compile time). To pomeni, da prevajalnik ve, koliko pomnilnika bo potreboval za shranjevanje te spremenljivke. Primeri takih spremenljivk so:

- **Primitivni tipi**: `i32`, `u64`, `f64`, `bool`, `char`
- **Tabela (array)**: Tabela ima fiksno velikost, ki je določena ob definiciji.

#### Primer:
```rust
fn main() {
    let stevilo: i32 = 42; // i32 ima fiksno velikost 4 bajte
    let resnica: bool = true; // bool ima fiksno velikost 1 bajt
    let znak: char = 'R'; // char ima fiksno velikost 4 bajte
    let tabela: [i32; 3] = [1, 2, 3]; // Tabela ima fiksno velikost 3 elemente

    println!("stevilo = {}, resnica = {}, znak = {}", stevilo, resnica, znak);
    println!("tabela = {:?}", tabela);
}
```

V tem primeru so vse spremenljivke fiksno velike, saj je njihova velikost znana že v času prevajanja.

---

### **Dinamično velike spremenljivke**
Dinamično velike spremenljivke imajo velikost, ki se lahko spreminja med izvajanjem programa (runtime). Njihova velikost ni znana v času prevajanja. Primeri takih spremenljivk so:

- **String**: Dinamično velik niz, ki lahko raste ali se skrči.
- **Vektor (Vec)**: Dinamično velik seznam elementov.
- **Druge zbirke**: Kot so `HashMap`, `HashSet` itd.

#### Primer:
```rust
fn main() {
    let mut niz = String::from("Pozdravljen"); // String je dinamično velik
    niz.push_str(", svet!"); // Niz lahko raste

    let mut vektor = vec![1, 2, 3]; // Vektor je dinamično velik
    vektor.push(4); // Vektor lahko raste

    println!("niz = {}", niz);
    println!("vektor = {:?}", vektor);
}
```

V tem primeru sta `niz` in `vektor` dinamično velika, saj se njihova velikost lahko spreminja med izvajanjem programa.

---

### **Ključne razlike**
| Značilnost               | Fiksno velike spremenljivke         | Dinamično velike spremenljivke       |
|--------------------------|-------------------------------------|--------------------------------------|
| **Velikost**             | Znana v času prevajanja            | Znana šele med izvajanjem programa   |
| **Primeri**              | `i32`, `bool`, `char`, `[T; N]`    | `String`, `Vec<T>`, `HashMap<K, V>` |
| **Skladiščenje**         | Na skladu (stack)                  | Na kopici (heap)                    |
| **Hitrost dostopa**      | Hitrejši dostop                    | Počasnejši dostop                   |
| **Fleksibilnost**        | Manj fleksibilne                   | Bolj fleksibilne                    |

---

### **Kdaj uporabiti fiksno velike spremenljivke?**
- Ko je velikost podatkov znana in se ne spreminja.
- Ko želite hitrejši dostop do podatkov.
- Ko delate s primitivnimi tipi ali majhnimi, fiksnimi zbirkami.

#### Primer:
```rust
fn main() {
    let koordinate: (f64, f64) = (46.0569, 14.5058); // Tuple s fiksno velikostjo
    println!("Koordinate Ljubljane: {:?}", koordinate);
}
```

---

### **Kdaj uporabiti dinamično velike spremenljivke?**
- Ko je velikost podatkov neznana ali se lahko spreminja.
- Ko delate z zbirkami, ki lahko rastejo ali se skrčijo.
- Ko potrebujete fleksibilnost.

#### Primer:
```rust
fn main() {
    let mut imena = Vec::new(); // Dinamično velik vektor
    imena.push("Ana");
    imena.push("Bojan");
    imena.push("Cvetka");

    println!("Imena: {:?}", imena);
}
```

---

### **Pomembno o lastništvu (ownership)**
V Rustu je pomnilniško varno delo z dinamično velikimi spremenljivkami zaradi sistema lastništva. Na primer:
- Ko prenesete lastništvo dinamično velike spremenljivke, se originalna spremenljivka ne more več uporabljati.
- Če želite deliti podatke, uporabite reference (`&`).

#### Primer:
```rust
fn main() {
    let niz1 = String::from("Zdravo"); // Dinamično velik niz
    let niz2 = niz1; // Lastništvo se prenese na niz2
    // println!("{}", niz1); // To bi povzročilo napako
    println!("{}", niz2);

    let niz3 = String::from("Svet");
    pozdravi(&niz3); // Posredujemo referenco
    println!("{}", niz3); // niz3 je še vedno veljaven
}

fn pozdravi(niz: &String) {
    println!("Pozdravljen, {}!", niz);
}
```

---

### **Zaključek**
- **Fiksno velike spremenljivke** so hitrejše in varno za uporabo, ko je velikost znana vnaprej.
- **Dinamično velike spremenljivke** so fleksibilne in omogočajo delo s podatki, katerih velikost se spreminja.

Z vajo in uporabo teh konceptov v praksi boš hitro obvladal Rust! 🚀

Če imaš kakršnakoli vprašanja, vprašaj! 😊
