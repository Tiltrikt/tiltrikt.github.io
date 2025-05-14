---
layout: default
title: Ako používať s inými brokermi
---

[<-- Na hlavnú stránku](index.md)
## Požiadavky na brokera

Broker musí podporovať **kompaktné témy** (compact topics). Sú to témy, ktoré pre daný kľúč uchovávajú len poslednú odoslanú hodnotu. Tento mechanizmus je kľúčový pre správnu funkčnosť Orionu pri použití iného brokera ako Kafka.

## Kroky pre vytvorenie adaptéra klienta pre iný broker

1. **Vytvorte nový modul pre adaptér**  
   V koreňovom adresári vášho projektu vytvorte nový modul s názvom `orion-client-<názov_brokera>-adapter`.

2. **Pridajte závislosti do konfiguračného súboru modulu**  
   Do konfiguračného súboru závislostí tohto nového modulu (napr. `build.gradle` alebo `pom.xml`) pridajte nasledujúce závislosti:

    - `api "dev.tiltrikt:orion-client-domain:1.0.1"`
    - `api "dev.tiltrikt:orion-client-common:1.0.1"`

3. **Implementujte Java rozhranie `EventPublisher`**  
   Rozhranie `EventPublisher` je definované v module `orion-client-common`. Vytvorte implementáciu tohto rozhrania v novom modulu. Táto implementácia bude zodpovedná za posielanie udalostí (ako registrácia, srdcový tep, odhlásenie) do vybraného brokera.

4. **Implementujte konzumenta pre spracovanie udalostí**  
   V novom modulu implementujte konzumenta, ktorý bude prijímať a spracovávať udalosti prichádzajúce z brokera. Minimálne musíte spracovať udalosti, ktoré oznamujú zmeny v registri, ako napríklad `RegistryUpdateEvent`. Tento konzument by mal:

    - Prijať príslušnú udalosť zo servera.
    - Z udalosti extrahovať informácie o inštanciách (napr. konvertovať ich na objekt `OrionInstance`).
    - Pomocou poskytnutého objektu `RegistryRepository` aktualizovať lokálnu evidenciu (cache) inštancií služieb v klientskej aplikácii, napríklad volaním metódy `save(OrionInstance)` pre každú zmenenú inštanciu.

## Záver

Implementácia nového adaptéra pre ne-Kafka broker v systéme Orion zahŕňa vytvorenie nového modulu, konfiguráciu závislostí, implementáciu publikovania udalostí a implementáciu konzumenta pre príjem a spracovanie udalostí. Po správnom nastavení bude váš klient schopný komunikovať s vybraným brokerom a spracovávať udalosti, ktoré oznamujú zmeny v systéme.
