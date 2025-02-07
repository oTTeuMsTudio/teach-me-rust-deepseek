Kako bi lahko zgradili projekt za delovni tok s tremi agenti (vodja projekta, arhitekt rešitev in razvijalec zaledja) v Rustu. Ker je to obsežen projekt, bomo razdelili proces na manjše, obvladljive korake in se osredotočili na ključne koncepte.

**1. Definiranje Agentov in Njihovih Nalog:**

Najprej definirajmo strukture za naše agente in njihove naloge. Uporabili bomo `struct` za predstavitev agentov in `enum` za različne vrste nalog.

```rust
#[derive(Debug)]
struct Agent {
    ime: String,
    vloga: Vloga,
}

#[derive(Debug)]
enum Vloga {
    VodjaProjektov,
    ArhitektResitev,
    RazvijalecZaledja,
}

#[derive(Debug)]
enum Naloga {
    NacrtovanjeArhitekture(String), // Opis arhitekture
    ImplementacijaFunkcionalnosti(String), // Opis funkcionalnosti
    UpravljanjeProjekta(String), // Opis projekta
}

// Primer ustvarjanja agentov
fn ustvari_agente() -> Vec<Agent> {
    vec![
        Agent { ime: "Ana".to_string(), vloga: Vloga::VodjaProjektov },
        Agent { ime: "Boštjan".to_string(), vloga: Vloga::ArhitektResitev },
        Agent { ime: "Cvetka".to_string(), vloga: Vloga::RazvijalecZaledja },
    ]
}
```

**2. Komunikacija Med Agenti (Sporočila):**

Agenti bodo komunicirali preko sporočil. Tudi tukaj lahko uporabimo `enum`:

Rust

```
#[derive(Debug)]
enum Sporocilo {
    NovaNaloga(Naloga),
    DokončanaNaloga(Naloga),
    ZahtevaZaInformacije(String),
    Informacije(String),
}
```

**3. Delovni Tok (Workflow):**

Tukaj je ključni del, kjer definiramo, kako poteka delovni tok. Uporabili bomo primer preprostega delovnega toka:

- Vodja projektov ustvari nalogo za načrtovanje arhitekture.
- Arhitekt rešitev prejme nalogo, jo izvede in pošlje sporočilo o dokončani nalogi.
- Vodja projektov ustvari nalogo za implementacijo funkcionalnosti.
- Razvijalec zaledja prejme nalogo, jo izvede in pošlje sporočilo o dokončani nalogi.

Rust

```
fn izvajaj_delovni_tok(agenti: &Vec<Agent>) {
    let vodja = najdi_agenta(&agenti, Vloga::VodjaProjektov).unwrap();
    let arhitekt = najdi_agenta(&agenti, Vloga::ArhitektResitev).unwrap();
    let razvijalec = najdi_agenta(&agenti, Vloga::RazvijalecZaledja).unwrap();

    // Vodja ustvari nalogo za arhitekta
    let naloga_arhitektura = Naloga::NacrtovanjeArhitekture("Spletna aplikacija za upravljanje projektov".to_string());
    println!("{}: Nova naloga: {:?}", vodja.ime, naloga_arhitektura);

    // ... (tukaj bi bila logika za pošiljanje sporočila arhitektu) ...

    // Simulacija, da je arhitekt dokončal nalogo
    let odgovor_arhitekt = Sporocilo::DokončanaNaloga(naloga_arhitektura);
    println!("{}: Dokončana naloga: {:?}", arhitekt.ime, odgovor_arhitekt);

    // ... (logika za obdelavo odgovora in ustvarjanje nove naloge za razvijalca) ...

    // ... (itd.) ...
}

fn najdi_agenta(agenti: &Vec<Agent>, vloga: Vloga) -> Option<&Agent> {
    agenti.iter().find(|agent| agent.vloga == vloga)
}
```

**4. Implementacija Logike za Pošiljanje Sporočil:**

V praksi bi uporabili kakšen mehanizem za pošiljanje sporočil med agenti (npr. kanali (`mpsc`), aktorski model, ali celo zunanji sistem za sporočila). Zaenkrat smo to izpustili za preprostost.

**5. Razširitve:**

- **Aktorji:** Za bolj kompleksen sistem bi bilo smiselno uporabiti aktorski model (npr. z uporabo crate-a `tokio::actor`). Vsak agent bi bil svoj aktor, ki bi asinhrono obdeloval sporočila.
- **Stanje:** Agenti bi lahko imeli svoje stanje, ki bi ga shranjevali in spreminjali glede na prejeta sporočila.
- **Trajnost:** Za shranjevanje stanja in nalog bi lahko uporabili bazo podatkov ali kakšen drug mehanizem za trajnost.
- **Testiranje:** Pomembno je, da napišemo teste za vsak del kode, da zagotovimo pravilno delovanje.

Ta primer je seveda zelo poenostavljen, vendar vam daje osnovno idejo, kako bi lahko začeli graditi tak projekt v Rustu. Ključno je, da razdelite problem na manjše dele in postopoma dodajate funkcionalnost. Če imate kakšna specifična vprašanja, me vprašajte!
