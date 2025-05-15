---
layout: default
title: Ako používať s inými brokermi
---

[<-- Na hlavnú stránku](index.md)
## Požiadavky na brokera

Broker musí podporovať **kompaktné témy** (*compact topics*). Ide o témy, ktoré pre daný kľúč uchovávajú iba poslednú odoslanú hodnotu. Tento mechanizmus je nevyhnutný pre správne fungovanie systému Orion pri použití iného brokera ako Kafka.

## Pridanie vlastných modulov do `settings.gradle`

Ak pridávate nové moduly (`orion-common-<názov_brokera>`, `orion-client-<názov_brokera>-adapter`, `orion-service-<názov_brokera>-adapter`), **nezabudnite ich zaregistrovať v súbore `settings.gradle`**.

### Kde to upraviť

V súbore `settings.gradle` sa nachádza zoznam všetkých modulov, ktoré sú súčasťou projektu:

```groovy
[
    "orion-common",
    "orion-common-event",
    ...
].forEach {
    include it
}
```
---

## Vytvorenie modulu s definíciou tém

1. **Vytvorte nový modul**  
   V koreňovom adresári projektu vytvorte nový modul s názvom `orion-common-<názov_brokera>`.

   2. **Definujte témy a ich konfiguráciu**  
      V tomto module definujte názvy tém a ich konfiguráciu špecifickú pre daný broker. Môžete vytvoriť triedu `BrokerTopicConfiguration`, ktorá bude obsahovať konštanty a konfigurácie tém podobne ako `KafkaTopicConfiguration`. Podrobnosti o nastavení tém na príklade Kafky nájdete na tomto odkaze   : [Návod na konfiguráciu tém](/kafka-topics.md).

---

## Kroky pre vytvorenie adaptéra klienta pre iný broker

1. **Vytvorte nový modul pre adaptér**  
   V koreňovom adresári projektu vytvorte nový modul s názvom `orion-client-<názov_brokera>-adapter`.

2. **Pridajte závislosti do konfiguračného súboru modulu**  
   Do konfiguračného súboru závislostí (napr. `build.gradle`) pridajte nasledujúce závislosti:

   ```groovy
   api "dev.tiltrikt:orion-client-domain:1.0.1"
   api "dev.tiltrikt:orion-client-common:1.0.1"
   api "<skupina>:orion-common-<názov_brokera>:<version>"
   ```

3. **Implementujte rozhranie `EventPublisher`**  
   Rozhranie `EventPublisher` je definované v module `orion-client-common`. Vytvorte implementáciu tohto rozhrania v novom modulu. Táto implementácia bude zodpovedná za posielanie udalostí (ako registrácia, srdcový tep, odhlásenie) do vybraného brokera.

4. **Implementujte príslušné konzumentské triedy na strane servera**<br>
   Server musí vedieť prijímať správy (prostredníctvom svojich konzumentov), ktoré klient odosiela.

## Kroky pre vytvorenie adaptéra servera pre iný broker

1. **Vytvorte nový modul pre adaptér**  
   V koreňovom adresári projektu vytvorte nový modul s názvom `orion-service-<názov_brokera>-adapter`.

2. **Pridajte závislosti do konfiguračného súboru modulu**  
      Do konfiguračného súboru závislostí tohto nového modulu (napr. `build.gradle`) pridajte nasledujúce závislosti:
```groovy
    api "org.springframework.boot:spring-boot-starter"
    api "dev.tiltrikt:orion-service-domain:1.0.1"
    api "dev.tiltrikt:orion-service-common:1.0.1"
    api "<skupina>:orion-common-<názov_brokera>:<version>"
```

3. **Implementujte rozhrania** `CandidateRequestPublisher`, `HeartbeatErrorPublisher`, `LeaderHeartbeatPublisher`, `RegistryUpdatePublisher`, `ReplicationRegistryUpdatePublisher`, `VotePublisher`.

4. **Implementujte príslušné konzumentské triedy na strane klienta**<br>
   Klient musí vedieť prijať správy, ktoré server posiela cez implementované publishery.

5. **Implementujte konzumentské triedy pre spracovanie udalostí z**:
   * `CandidateRequestPublisher`
   * `LeaderHeartbeatPublisher`
   * `VotePublisher`
   * `ReplicationRegistryUpdatePublisher`

6. **Implementujte rozhrania `ConsumerLeadershipManager` a `ConsumerManager`** na riadenie stavu Kafka konzumentov v závislosti od roly uzla.

```
Keď sa uzol stane Líderom:
  Zastaviť príjem (počúvanie) replikačných udalostí
  Spustiť príjem (počúvanie) udalostí inštancií

Keď sa uzol stane Followerom:
  Zastaviť príjem (počúvanie) udalostí inštancií
  Spustiť príjem (počúvanie) replikačných udalostí
```