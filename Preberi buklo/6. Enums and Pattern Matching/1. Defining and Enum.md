Kako definirati enume (enumeracije), vrednosti enumov in "option enum" ter njegove prednosti pred uporabo `null` vrednosti?

## **Kaj je Enum?**

Enum (enumeracija) je podatkovni tip, ki predstavlja eno od več možnih variant. Predstavljaj si ga kot nekaj, kar lahko ima več različnih "stanj" ali "tipov". V Rustu so enumi zelo močni in se pogosto uporabljajo za predstavljanje različnih možnosti ali stanj v tvojem programu.

## **Definiranje Enuma**

Definirajmo preprost enum, ki predstavlja barvo:

```rust
enum Barva {
    Rdeča,
    Zelena,
    Modra,
}
```

Tukaj smo definirali enum `Barva`, ki ima tri variante: `Rdeča`, `Zelena` in `Modra`. Vsaka od teh variant je vrednost enuma.

## **Uporaba Enum Vrednosti**

Enum vrednosti lahko uporabljaš tako:

```rust
let moja_barva = Barva::Zelena;

match moja_barva {
    Barva::Rdeča => println!("Barva je rdeča."),
    Barva::Zelena => println!("Barva je zelena."),
    Barva::Modra => println!("Barva je modra."),
}
```

V tem primeru smo ustvarili spremenljivko `moja_barva`, ki ima vrednost `Barva::Zelena`. Uporabili smo `match` izraz, da preverimo, katera varianta enuma je `moja_barva` in ustrezno izvedemo kodo.

## **Enum z Dodatnimi Podatki**

Enumi v Rustu so lahko še bolj zmogljivi, saj lahko vsaka varianta vsebuje dodatne podatke. Poglejmo primer, kjer imamo enum, ki predstavlja različne oblike:

```rust
enum Oblika {
    Krog(f64), // Polmer kroga
    Pravokotnik(f64, f64), // Širina in višina pravokotnika
    Trikotnik(f64, f64, f64), // Stranice trikotnika
}

let moja_oblika = Oblika::Krog(5.0);

match moja_oblika {
    Oblika::Krog(polmer) => println!("Krog s polmerom {}", polmer),
    Oblika::Pravokotnik(sirina, visina) => println!("Pravokotnik s širino {} in višino {}", sirina, visina),
    Oblika::Trikotnik(a, b, c) => println!("Trikotnik s stranicami {}, {} in {}", a, b, c),
}
```

Tukaj vidimo, da `Oblika::Krog` vsebuje polmer (tip `f64`), `Oblika::Pravokotnik` vsebuje širino in višino, `Oblika::Trikotnik` pa tri stranice.

## **Option Enum in Prednosti pred Null**

V Rustu ne obstaja `null` kot ga poznamo iz drugih jezikov (npr. Java, C++). Namesto tega imamo `Option` enum, ki predstavlja možnost, da vrednost obstaja ali ne.

```rust
enum Option<T> {
    Some(T),
    None,
}
```

`Option<T>` je generični enum. `T` predstavlja tip vrednosti, ki jo lahko vsebuje. `Some(T)` pomeni, da vrednost obstaja, `None` pa pomeni, da vrednosti ni.

## **Zakaj je Option boljši od Null?**

- **Eksplicitnost:** `Option` te prisili, da izrecno preveriš, ali vrednost obstaja (`Some`) ali ne (`None`). S tem se izogneš napakam, ki nastanejo, če pozabiš preveriti, ali je vrednost `null`.
- **Tipna varnost:** `Option` je tip, kar pomeni, da ga lahko uporabljaš varno in da te kompilator opozori, če poskušaš uporabiti vrednost, ki morda ne obstaja. `null` ni tip, zato ga lahko uporabljaš povsod, kar lahko vodi do napak.
- **Čitljivost kode:** Uporaba `Option` naredi kodo bolj čitljivo, saj je jasno, da vrednost morda ne obstaja.

**Primer Uporabe Option**

```rust
fn poišči_uporabnika(id: i32) -> Option<String> {
    // ... poišče uporabnika z danim ID-jem ...
    if id == 1 {
        Some("Janez".to_string())
    } else {
        None
    }
}

let ime_uporabnika = poišči_uporabnika(1);

match ime_uporabnika {
    Some(ime) => println!("Uporabnikovo ime je {}", ime),
    None => println!("Uporabnik ni bil najden."),
}

let drugo_ime = poišči_uporabnika(2);
println!("{:?}", drugo_ime); // Izpisal bo None
```

V tem primeru funkcija `poišči_uporabnika` vrača `Option<String>`. Če uporabnik obstaja, vrne `Some(ime)`, sicer vrne `None`. S tem smo zagotovili, da moramo preveriti, ali uporabnik obstaja, preden ga uporabimo.

# Qwen razlaga

### 1. Definicija `enum` v Rustu

`Enum` (oz. enumeracija) je podatkovni tip v Rustu, ki omogoča določitev skupine možnih vrednosti. Vsaka izmed teh vrednosti se imenuje "varianta" ali "člen". Enumeracije so zelo uporabne za predstavitev različnih stanj ali možnosti v programu.

#### Primer:
```rust
enum Dan {
    Pon,
    Tor,
    Sre,
    Cet,
    Pet,
    Sob,
    Ned,
}
```

V tem primeru smo definirali `enum` z imenom `Dan`, ki ima sedem variant: `Pon`, `Tor`, `Sre`, `Čet`, `Pet`, `Sob` in `Ned`.

### 2. Uporaba `enum` vrednosti

Vsako od teh variant lahko uporabimo kot vrednost spremenljivke:

```rust
fn main() {
    let dan = Dan::Pon; // Določimo, da je dan "Ponedeljek"
    
    match dan {
        Dan::Pon => println!("Danes je ponedeljek."),
        Dan::Tor => println!("Danes je torek."),
        Dan::Sre => println!("Danes je sreda."),
        Dan::Cet => println!("Danes je četrtek."),
        Dan::Pet => println!("Danes je petek."),
        Dan::Sob => println!("Danes je sobota."),
        Dan::Ned => println!("Danes je nedelja."),
    }
}
```

### 3. Option Enum

Rust ima poseben `enum`, imenovan `Option`, ki ga uporabljamo za predstavitev vrednosti, ki jo lahko vrne funkcija ali pa ne. To je bolj varno od uporabe `null` v drugih jezikih, ker nam pravi kompajler, da moramo obravnavati primer, ko ni vrednosti.

#### Definicija Option:
```rust
enum Option<T> {
    Some(T),
    None,
}
```

- `Some(T)` pomeni, da obstaja vrednost tipa `T`.
- `None` pomeni, da vrednosti ni.

#### Primer uporabe Option:
```rust
fn deljenje(Stevec: i32, imenovalca: i32) -> Option<i32> {
    if imenovalca == 0 {
        None // Ni mogoče deliti z 0
    } else {
        Some(stevec / imenovalca)
    }
}

fn main() {
    let rezultat = deljenje(10, 2);
    
    match rezultat {
        Some(vrednost) => println!("Rezultat je {}", vrednost),
        None => println!("Deljenje ni mogoče."),
    }
}
```

### 4. Prednosti `Option` obrazca pred `null` vrednostmi

- **Varnost**: V Rustu ne morete neposredno uporabiti `null` vrednosti. Če želite predstaviti odsotnost vrednosti, morate uporabiti `Option`. To pomeni, da vas kompajler prisili, da obravnavate oba možna scenarija (`Some` in `None`), kar preprečuje napake, ki bi lahko prišle zaradi neprepoznanih `null` vrednosti.
  
- **Jasnost**: Ko uporabljate `Option`, je jasno, da je morda mogoče, da vrednosti ni. To pomeni, da si programerjozi večjo ozaveščenost o potencialnih problemih v kodi.

- **Preventivna programiranje**: Ker morate uporabiti `match` ali drugo obliko vzorec-vezanja za dostop do vsebine `Option`, ste prepričani, da boste ustrezno reagirali na primer, ko ni vrednosti.
