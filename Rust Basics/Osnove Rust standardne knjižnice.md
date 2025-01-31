Naučimo se osnov standardne knjižnice (Std Library) v programskem jeziku Rust. Rust je sistemski programski jezik, ki poudarja varnost, hitrost in sočasnost. Standardna knjižnica Rust ponuja številne uporabne module in funkcije, ki olajšajo razvoj aplikacij.

### Osnove Standardne Knjižnice v Rust-u

1. **`std::fmt`** - Formatiranje izpisa
   - `println!` - Izpiše vrednosti v konzolo.
   ```rust
   println!("Pozdravljen, svet!");
   ```

2. **`std::vec::Vec`** - Dinamični seznami
   - `Vec` je dinamični seznam, ki lahko raste ali se skrči.
   ```rust
   let mut seznam = Vec::new();
   seznam.push(1);
   seznam.push(2);
   println!("{:?}", seznam); // Izpiše: [1, 2]
   ```

3. **`std::collections::HashMap`** - Slovarji
   - `HashMap` je zbirka parov ključ-vrednost.
   ```rust
   use std::collections::HashMap;

   let mut slovar = HashMap::new();
   slovar.insert("ključ", "vrednost");
   println!("{:?}", slovar.get("ključ")); // Izpiše: Some("vrednost")
   ```

4. **`std::option::Option`** - Opcijske vrednosti
   - `Option` je enum, ki predstavlja morebitno prisotno vrednost (`Some`) ali odsotnost vrednosti (`None`).
   ```rust
   let nekaj = Some(5);
   match nekaj {
       Some(x) => println!("Vrednost: {}", x),
       None => println!("Ni vrednosti"),
   }
   ```

5. **`std::result::Result`** - Rezultati operacij
   - `Result` je enum, ki predstavlja uspeh (`Ok`) ali napako (`Err`).
   ```rust
   let rezultat: Result<i32, &str> = Ok(10);
   match rezultat {
       Ok(vrednost) => println!("Uspeh: {}", vrednost),
       Err(napaka) => println!("Napaka: {}", napaka),
   }
   ```

6. **`std::fs`** - Delo z datotekami
   - Modul `fs` omogoča branje in pisanje datotek.
   ```rust
   use std::fs;

   let vsebina = fs::read_to_string("pot/do/datoteke.txt").expect("Napaka pri branju datoteke");
   println!("{}", vsebina);
   ```

7. **`std::thread`** - Sočasnost
   - Modul `thread` omogoča ustvarjanje in upravljanje niti.
   ```rust
   use std::thread;

   let roka = thread::spawn(|| {
       println!("Pozdravljen iz niti!");
   });

   roka.join().unwrap();
   ```

8. **`std::net`** - Omrežna komunikacija
   - Modul `net` omogoča delo z omrežnimi vtičnicami.
   ```rust
   use std::net::TcpListener;

   let poslušalec = TcpListener::bind("127.0.0.1:8080").unwrap();
   for tok in poslušalec.incoming() {
       let tok = tok.unwrap();
       println!("Nova povezava!");
   }
   ```

### Zaključek

To so le osnove standardne knjižnice Rust. Rust-ova standardna knjižnica je zelo obsežna in ponuja številne druge module in funkcije za različne namene. Za bolj podrobno razumevanje priporočam, da si ogledate uradno dokumentacijo Rust: [Rust Documentation](https://doc.rust-lang.org/std/).
