Tukaj se bomo osredotočili na strukturo `User` in implementacijo (`impl`) metod zanjo. Rust je odličen jezik za organizacijo podatkov in funkcionalnosti s pomočjo struktur in metod.

### 1. Definicija strukture `User`

Struktura (`struct`) je zbirka podatkov, ki jih združimo v eno entiteto. V tem primeru bomo ustvarili strukturo `User`, ki bo predstavljala uporabnika.

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

Tukaj smo definirali strukturo `User` s štirimi polji:

- `username`: Uporabniško ime (tip `String`).
- `email`: E-poštni naslov (tip `String`).
- `sign_in_count`: Število prijav (tip `u64`).
- `active`: Ali je uporabniški račun aktiven (tip `bool`).

---

### 2. Ustvarjanje instance `User`

Ko imamo strukturo, lahko ustvarimo njeno instanco (objekt).

```rust
let uporabnik = User {
    username: String::from("janeznovak"),
    email: String::from("janez@example.com"),
    sign_in_count: 1,
    active: true,
};
```

Tukaj smo ustvarili instanco `User` z uporabniškim imenom "janeznovak", e-poštnim naslovom "janez@example.com", številom prijav 1 in aktivnim računom.

---

### 3. Dostop do polj strukture

Do posameznih polj strukture lahko dostopamo z uporabo pike (`.`).

```rust
println!("Uporabniško ime: {}", uporabnik.username);
println!("E-pošta: {}", uporabnik.email);
println!("Število prijav: {}", uporabnik.sign_in_count);
println!("Aktiven: {}", uporabnik.active);
```

---

### 4. Uporaba `impl` za metode

Z `impl` lahko definiramo metode, ki delujejo na strukturah. Metode so funkcije, ki so povezane s strukturo.

#### Primer metod za `User`:

```rust
impl User {
    // Metoda za pozdrav uporabnika
    fn pozdravi(&self) {
        println!("Pozdravljen, {}!", self.username);
    }

    // Metoda za preverjanje, ali je uporabnik aktiven
    fn je_aktiven(&self) -> bool {
        self.active
    }

    // Metoda za povečanje števila prijav
    fn prijavi_se(&mut self) {
        self.sign_in_count += 1;
    }
}
```

Tukaj smo definirali tri metode:

1. **`pozdravi`**: Izpiše pozdrav z uporabniškim imenom.
2. **`je_aktiven`**: Vrne `true`, če je uporabniški račun aktiven, sicer `false`.
3. **`prijavi_se`**: Poveča število prijav za 1. Ta metoda spreminja strukturo, zato uporabljamo `&mut self`.

---

### 5. Klic metod

Metode lahko kličemo na instancah strukture.

```rust
let mut uporabnik = User {
    username: String::from("truenjenir"),
    email: String::from("truenjenir@example.com"),
    sign_in_count: 5,
    active: true,
};

uporabnik.pozdravi(); // Izpiše: Pozdravljen, truenjenir!
println!("Je aktiven? {}", uporabnik.je_aktiven()); // Izpiše: Je aktiven? true

uporabnik.prijavi_se(); // Povečamo število prijav
println!("Število prijav: {}", uporabnik.sign_in_count); // Izpiše: Število prijav: 6
```

---

### 6. Asociirane funkcije

Poleg metod lahko v `impl` bloku definiramo tudi **asociirane funkcije**, ki niso vezane na instanco strukture. Te funkcije se običajno uporabljajo za ustvarjanje novih instanc.

#### Primer asociirane funkcije:

```rust
impl User {
    // Asociirana funkcija za ustvarjanje novega uporabnika
    fn nov(username: String, email: String) -> User {
        User {
            username,
            email,
            sign_in_count: 0,
            active: true,
        }
    }
}
```

Tukaj smo definirali asociirano funkcijo `nov`, ki ustvari novo instanco `User` z danim uporabniškim imenom in e-poštnim naslovom. Število prijav je nastavljeno na 0, račun pa je privzeto aktiven.

#### Uporaba asociirane funkcije:

```rust
let nov_uporabnik = User::nov(String::from("markoz"), String::from("marko@example.com"));
println!("Nov uporabnik: {}", nov_uporabnik.username); // Izpiše: Nov uporabnik: markoz
```

---

### 7. Celoten primer

Tukaj je celoten primer kode:

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

impl User {
    // Asociirana funkcija za ustvarjanje novega uporabnika
    fn nov(username: String, email: String) -> User {
        User {
            username,
            email,
            sign_in_count: 0,
            active: true,
        }
    }

    // Metoda za pozdrav uporabnika
    fn pozdravi(&self) {
        println!("Pozdravljen, {}!", self.username);
    }

    // Metoda za preverjanje, ali je uporabnik aktiven
    fn je_aktiven(&self) -> bool {
        self.active
    }

    // Metoda za povečanje števila prijav
    fn prijavi_se(&mut self) {
        self.sign_in_count += 1;
    }
}

fn main() {
    let mut uporabnik = User::nov(String::from("truenjenir"), String::from("truenjenir@example.com"));

    uporabnik.pozdravi(); // Izpiše: Pozdravljen, anaprek!
    println!("Je aktiven? {}", uporabnik.je_aktiven()); // Izpiše: Je aktiven? true

    uporabnik.prijavi_se(); // Povečamo število prijav
    println!("Število prijav: {}", uporabnik.sign_in_count); // Izpiše: Število prijav: 1
}
```

---

### 8. Zaključek

Sedaj znamo:

- definirati strukturo `User`.
- ustvariti instance strukture.
- dostopati do polj strukture.
- uporabiti `impl` za definiranje metod in asociiranih funkcij.

Strukture in metode so orodje za organizacijo podatkov in funkcionalnosti.
