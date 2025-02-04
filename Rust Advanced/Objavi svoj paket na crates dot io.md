Naučimo se, kako objaviti pakete na **crates.io**, kar je uradni registr paketov za nizkonivosjki programski jezik Rust.
### 1. **Priprava projekta**
Preden lahko objavimo paket, moramo imeti Rust projekt, ki ga želimo deliti. Če še nimaš projekta, ga ustvari z ukazom:

```bash
cargo new ime_tvojega_paketa
```

To bo ustvarilo novo mapo z osnovno strukturo projekta.

---

### 2. **Nastavitev `Cargo.toml`**
V mapi projekta poišči datoteko `Cargo.toml`. To je konfiguracijska datoteka, ki vsebuje metapodatke o tvojem paketu. Poskrbi, da so vsa polja pravilno izpolnjena:

```toml
[package]
name = "ime_tvojega_paketa"  # Ime paketa (mora biti edinstveno na crates.io)
version = "0.1.0"            # Različica paketa
edition = "2021"             # Različica Rusta
authors = ["Tvoje Ime <tvoj.email@example.com>"]
description = "Kratek opis tvojega paketa"
license = "MIT"              # Izberi primerno licenco
repository = "https://github.com/tvoj_uporabnik/tvoj_repo"  # Neobvezno
```

---

### 3. **Preveri kodo**
Preden objaviš paket, preveri, ali je tvoja koda brez napak. To lahko narediš z ukazom:

```bash
cargo build
cargo test
```

Če je vse v redu, lahko nadaljuješ.

---

### 4. **Ustvari račun na crates.io**
Če še nimaš računa na [crates.io](https://crates.io), ga ustvari. Nato pridobi svoj **API token**:

1. Obišči [crates.io](https://crates.io) in se prijavi.
2. Pojdi v svoj profil in klikni na "Account Settings".
3. Ustvari nov API token.

---

### 5. **Prijava s `cargo`**
Svoj API token uporabi za prijavo v orodje `cargo`. Odpri terminal in vnesi:

```bash
cargo login tvoj_api_token
```

To bo shranilo tvoj token lokalno in omogočilo objavo paketov.

---

### 6. **Objavi paket**
Ko je vse pripravljeno, lahko objaviš paket z ukazom:

```bash
cargo publish
```

Če je vse v redu, bo tvoj paket objavljen na [crates.io](https://crates.io) in drugi razvijalci ga bodo lahko uporabljali.

---

### 7. **Posodabljanje paketa**
Če želiš posodobiti svoj paket, spremeni različico v `Cargo.toml` (npr. iz `0.1.0` v `0.1.1`) in ponovno izvedi:

```bash
cargo publish
```

---

### Dodatni nasveti
- **Ime paketa**: Ime mora biti edinstveno na crates.io. Če je že zasedeno, boš moral izbrati drugo ime.
- **Dokumentacija**: Dodaj dokumentacijo s komentarji `///` nad funkcijami in moduli. To bo prikazano na [docs.rs](https://docs.rs).
- **Testiranje**: Preveri, da tvoj paket deluje pravilno, preden ga objaviš.
