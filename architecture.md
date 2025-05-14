---
layout: default
title: Architektúra
---

[<-- Na hlavnú stránku](index.md)

# Architecture
```
.
└── orion/
    ├── orion-client-common
    ├── orion-client-domain
    ├── orion-client-kafka-adapter
    ├── orion-client-kafka-spring-boot-starter
    ├── orion-common
    ├── orion-common-event
    ├── orion-common-kafka
    ├── orion-service-common
    ├── orion-service-domain
    ├── orion-service-kafka-adapter
    └── orion-service-kafka-spring-boot-starter
```

## Moduly systému Orion

Systém Orion je rozdelený do viacerých modulov, ktoré združujú súvisiace funkcionality a zjednodušujú vývoj a údržbu. Moduly sú rozdelené na spoločné (common), klientské a serverové časti.

## Spoločné moduly (Common)

Tieto moduly obsahujú kód a definície zdieľané medzi klientskou a serverovou časťou systému.

### `orion-common`
Modul obsahuje zdieľanú triedu (model), ktorá definuje **stav inštancie** a zabezpečuje tak jeho konzistentnú reprezentáciu a spracovanie na strane klienta aj servera.

| Stav      | Význam                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| :-------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `UP`      | Inštancia je plne funkčná, aktívna a pripravená spracúvať požiadavky.                                                                                                                                                                                                                                                                                                                                                                                     |
| `DOWN`    | Inštancia nie je aktívna, nie je dostupná alebo nie je pripravená prijímať požiadavky (napr. bola zastavená, je nedostupná v sieti, nastala v nej chyba, alebo ju systém označil ako neaktívnu).                                                                                                                                                                                                                                                         |
| `UNKNOWN` | Tento stav indikuje, že aktuálny stav inštancie nie je jednoznačne známy a systém ho momentálne overuje. Nastáva napríklad, ak inštancia po dlhšej neaktivite (ktorá presiahla čas povolený pre neaktivitu - `lease-duration`) odošle srdcový tep. Server vtedy nastaví jej stav na `UNKNOWN` a iniciuje overovací proces, aby zistil, či je inštancia skutočne aktívna, alebo či išlo len o oneskorený (už neplatný) signál o jej predchádzajúcom stave. |

---

### `orion-common-event`
Tento modul obsahuje **definície udalostí (eventov)**, ktoré sa používajú na výmenu informácií medzi komponentmi systému prostredníctvom brokera (napr. Kafka).

| Udalosť                       | Význam                                                                                                                                                                                     |
| :---------------------------- |:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `InstanceDeregistrationEvent` | Inštancia túto udalosť odošle, aby informovala systém (Orion server), že sa riadne odhlasuje a ukončuje svoju činnosť.                                                  |
| `InstanceHeartbeatEvent`      | Inštancia ju posiela v pravidelných intervaloch, aby signalizovala, že je stále aktívna a dostupná (slúži ako „srdcový tep“).                                                              |
| `InstanceRegistrationEvent`   | Inštancia odošle túto udalosť, keď sa spúšťa a je pripravená pracovať. Server na základe nej inštanciu zaregistruje a informuje ostatné komponenty o jej dostupnosti.                      |
| `NodeHeartbeatEvent`          | Podobný ako `InstanceHeartbeatEvent`, ale potvrdzuje aktivitu a dostupnosť samotného serverového uzla Orion.                                                                               |
| `RegistryUpdateEvent`         | Server ho odošle ostatným systémom alebo inštanciám, ktoré potrebujú vedieť o zmenách v registri (napr. o pridaní, odstránení alebo zmene stavu inštancií).                                |
| `ReplicationEvent`            | Udalosť slúžiaca na replikáciu údajov registrata (informácií o inštanciách) medzi viacerými uzlami Orion servera, aby sa zabezpečila konzistencia a dostupnosť informácií v celom klastri. |

---

### `orion-common-kafka`
Tento modul obsahuje **konfiguráciu tém Kafka** a súvisiace nastavenia potrebné pre komunikáciu cez Kafka broker. Viac informácií o konfigurácii tém nájdete na tomto odkaze: [Návod na konfiguráciu tém](/kafka-topics.md).

**Externé závislosti:**
* `org.springframework.kafka:spring-kafka`: Zabezpečuje **integráciu** aplikácie s Apache Kafka v prostredí Spring Boot.
* `com.fasterxml.jackson.*`: Tieto knižnice sa používajú na **serializáciu a deserializáciu** Java objektov do/z formátu JSON, čo je nevyhnutné na správne spracovanie dátových typov prenášaných cez Kafka správy.

## Klientské moduly (Client)

Tieto moduly obsahujú kód a logiku spúšťanú na strane inštancie nejakej služby, ktorá sa chce zaregistrovať do Orion systému a využívať služby objavovania.

### `orion-client-common`
Tento modul deklaruje **základné API (rozhranie)** pre klienta Orion systému. Obsahuje jedno rozhranie `EventPublisher`, ktoré definuje metódy na odosielanie kľúčových udalostí týkajúcich sa životného cyklu inštancie:
* `publishRegistration(InstanceRegistrationEvent event)`: Na informovanie centrálneho registra o spustení novej inštancie.
* `publishDeregistration(InstanceDeregistrationEvent event)`: Na informovanie centrálneho registra o ukončení činnosti inštancie.
* `publishHeartbeat(InstanceHeartbeatEvent event)`: Na posielanie srdcového tepu inštancie.

---

### `orion-client-domain`
Tento modul obsahuje **hlavnú logiku klientskej aplikácie**. Je zodpovedný za správu životného cyklu inštancie v systéme (registrácia, srdcové tepy) a za distribuovanie informácií o aktívnych inštanciách po celom systéme.Okrem toho obsahuje logiku výberu lídra a replikácie údajov.

**Externé závislosti:**
* `org.springframework.cloud:spring-cloud-commons`: Poskytuje spoločné abstrakcie a základnú logiku pre Spring Cloud projekty, čím zabezpečuje kompatibilitu.
* `org.springframework.cloud:spring-cloud-starter-loadbalancer` a `org.springframework.boot:spring-boot-starter-webflux` - zabezpečujú kompatibilitu s Spring Cloud Gateway

**Vnútorná štruktúra (Balíky):**

* **`job`**:  
  Obsahuje naplánované úlohy. Pozostáva z jednej triedy `HeartbeatJob`, ktorá sa stará o **pravidelné odosielanie srdcových tepov**.

* **`client`**:  
  Obsahuje hlavné klientské triedy pre prístup k informáciám o službách:
  - `OrionDiscoveryClient`: Implementuje Spring rozhranie `DiscoveryClient` pre **synchrónny** prístup.
  - `OrionReactiveDiscoveryClient`: Implementuje Spring rozhranie `ReactiveDiscoveryClient` pre **asynchrónny, neblokujúci (reaktívny)** prístup.

* **`repository`**:  
  Zabezpečuje **lokálne ukladanie a správu informácií** o inštanciách služieb na strane klienta.

* **`registration`**:  
  Obsahuje triedy zodpovedné za **automatickú registráciu** klientskej inštancie do systému objavovania pri jej štarte.

* **`instance`**:  
  Obsahuje kľúčovú triedu `OrionInstance`, ktorá **reprezentuje jednu bežiacu inštanciu služby**. Uchováva jej ID, názov, stav (`UP`, `DOWN`, `UNKNOWN`) a ďalšie metadáta, pričom implementuje Spring rozhranie `Registration`.

---

### `orion-client-kafka-adapter`
Tento modul obsahuje **špecifickú implementáciu komunikácie** pre prácu s Kafka brokerom na strane klienta. Zahŕňa implementáciu Kafka konzumenta a producenta, ktoré posielajú a prijímajú udalosti definované v `orion-common-event`. Ak by sa použil iný typ brokera, vytvoril by sa nový modul adaptéra (napr. `orion-client-rabbitmq-adapter`).

---

### `orion-client-kafka-spring-boot-starter`
Tento modul funguje ako **Spring Boot Starter** pre klientskú časť systému Orion s podporou Kafky. Poskytuje **automatickú konfiguráciu**, ktorá umožňuje rýchlu a jednoduchú integráciu Orion klienta do ľubovoľnej Spring Boot aplikácie pridaním jedinej závislosti.

**Externé závislosti:**
* `org.springframework.boot:spring-boot-configuration-processor`: Automatizuje generovanie metadát o konfiguračných parametroch aplikácie.

## Serverové moduly (Server)

Tieto moduly obsahujú kód a logiku bežiacu na serverovej strane, ktorá spravuje centrálny register inštancií, a spracúva klientské udalosti.

### `orion-server-domain`
**Externé závislosti:**
* `org.springframework.statemachine:spring-statemachine-core`: Slúži na riadenie procesu výberu lídra.
* `org.springframework.boot:spring-boot-starter-data-jpa`: Umožňuje jednoduchý prístup k relačnej databáze pomocou Spring Data a štandardu JPA.
* `com.h2database:h2`: Samotná relačná databáza H2.

**Vnútorná štruktúra (Balíky):**

* **`statemachine`**: Obsahuje **stavy a udalosti** používané v stavovom automate riadiacom správanie uzlov v distribuovanom systéme. Stavy, v ktorých sa uzol môže nachádzať, sú uvedené nižšie:

| Stav        | Popis                                                                                            |
|-------------|--------------------------------------------------------------------------------------------------|
| `LEADER`    | Uzol v tomto stave vykonáva riadiace operácie a zabezpečuje replikáciu dát ostatným uzlom.       |
| `CANDIDATE` | Keď líder prestane odpovedať, ostatné uzly po náhodne zvolenom čase prechádzajú do tohto stavu, aby sa mohli stať novým lídrom.                                                                   |
| `FOLLOWER`  | Uzol, ktorý iba sleduje lídra a prijíma od neho replikované informácie.                          |

<br>

* **`job`**: Obsahuje **naplánované úlohy**, ktoré vykonávajú dôležité činnosti v systéme v pravidelných intervaloch. Obsahuje nasledujúce triedy:

  * **`LeaderHeartbeatJob`**: Pravidelne odosiela srdcový tep od aktuálneho lídra ostatným uzlom, aby potvrdil, že je stále aktívny.
  * **`LeaseExpirationCheckJob`**: Kontroluje, či registrácie služieb (inštancií) nevypršali. Ak áno, dané inštancie sa odstránia zo systému.
  * **`LeaderHeartbeatJobManager`** a **`LeaseExpirationCheckJobManager`**: Zabezpečujú spustenie a zastavenie plánovaných úloh v závislosti od stavu uzla. Napríklad LeaderHeartbeatJob sa spustí iba vtedy, keď uzol prejde do stavu líder.
  
<br>

* **`model`**: Obsahuje **dátové modely**, ktoré reprezentujú aktuálny stav jednotlivých uzlov a služieb v systéme. Obsahuje nasledujúce triedy:

  * **`InstanceModel`**: JPA entita reprezentujúca jednu inštanciu služby. Obsahuje údaje ako ID služby, host, port, metadáta, stav, čas expirácie atď.
  * **`OrionServiceNode`**: Model reprezentujúci konkrétny uzol systému. Obsahuje identifikátor, informáciu o tom, či je uzol lídrom a aktuálne číslo volebného obdobia.

<br>

* **`task`**: Obsahuje **časové úlohy súvisiace s voľbou lídra** v rámci Raft algoritmu. Obsahuje nasledujúce triedy:

  * **`ElectionTimeoutTask`**: Časovaná úloha, ktorá sa spustí po náhodne zvolenom čase. Po uplynutí času vyšle do stavového automatu udalosť `ELECTION_TIMEOUT`, čím sa spustí proces voľby nového lídra.
  * **`ElectionTimeoutTaskManager`**: Správca pre `ElectionTimeoutTask`. Jeho úlohou je reštartovať časovač, ktorý spustí úlohu `ElectionTimeoutTask` s novým náhodným oneskorením. Tým sa zabezpečí, že každý uzol čaká odlišný čas predtým, než sa pokúsi stať kandidátom.

<br>

* **`handler`**: Obsahuje **hlavnú logiku správania uzlov** v závislosti od ich stavu (`FOLLOWER`, `LEADER`). Implementuje správanie požadované pre jednotlivé fázy Raft algoritmu.


---

### `orion-server-kafka-adapter`
Tento modul obsahuje **špecifickú implementáciu komunikácie** pre prácu s **Kafka brokerom** na strane servera. Jeho úlohou je prijímať klientské udalosti (ako registrácie, srdcové tepy, odhlásenia) z Kafky a odosielať serverové udalosti (ako aktualizácie registry, udalosti voľby lídra, replikačné udalosti) do Kafky. Predstavuje pre serverovú logiku adaptér pre Kafka komunikáciu. Ak by sa použil iný typ brokera, vytvoril by sa nový modul adaptéra (napr. orion-server-rabbitmq-adapter).

---

### `orion-server-spring-boot-starter`
Tento modul funguje ako **Spring Boot Starter** pre serverovú časť systému Orion s podporou Kafky. Poskytuje **automatickú konfiguráciu**, ktorá umožňuje rýchlo a jednoducho nastaviť a spustiť Orion server, ako súčasť Spring Boot aplikácie pridaním jedinej závislosti. Zjednodušuje konfiguráciu Kafka konzumentov, producentov a ostatných častí.

**Externé závislosti:**
* `org.springframework.boot:spring-boot-configuration-processor`: Automatizuje generovanie metadát o konfiguračných parametroch aplikácie.


| Modul                                     | Účel                                                   | Závislosti                                                                                                                                                                                                                                                                                                |
| ----------------------------------------- | ------------------------------------------------------ |-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `orion-common`                            | Základné typy a nástroje zdieľané klientom aj serverom | —                                                                                                                                                                                                                                                                                                         |
| `orion-common-event`                      | Definície udalostí, ktoré sa prenášajú cez Kafka       | `orion-common`                                                                                                                                                                                                                                                                                            |
| `orion-common-kafka`                      | Konfigurácie Kafka klienta a common Kafka logika       | `org.springframework.kafka:spring-kafka` a `com.fasterxml.jackson...` aby Kafka správne pracovala s rôznymi typmi údajov                                                                                                                                                                                  |
| `orion-client-common`                     | Deklaruje API pre klienta                | `orion-common-event`                                                                                                                                                                                                                                                                                      |
| `orion-client-domain`                     | Hlavná logika klienta                      | `orion-client-common`, `org.springframework.cloud:spring-cloud-commons` pre zabezpečenie kompatibility s ostatnými službami spring cloud, `org.springframework.cloud:spring-cloud-starter-loadbalancer` a `org.springframework.boot:spring-boot-starter-webflux` pre kompatibilitu s Spring Cloud Gateway |
| `orion-client-kafka-adapter`              | Implementácia adaptera pre Kafka                       | `orion-client-domain`, `orion-common-kafka`                                                                                                                                                                                                                                                               |
| `orion-client-kafka-spring-boot-starter`  | Spring Boot štartér pre integráciu klienta s Kafka     | `orion-client-kafka-adapter`, `spring-boot-starter`                                                                                                                                                                                                                                                       |
| `orion-service-common`                    | Zdieľaná logika pre služby                             | `orion-common`, `orion-common-event`                                                                                                                                                                                                                                                                      |
| `orion-service-domain`                    | Doménová logika serverovej časti                       | `orion-service-common`                                                                                                                                                                                                                                                                                    |
| `orion-service-kafka-adapter`             | Kafka adapter pre server                               | `orion-service-domain`, `orion-common-kafka`                                                                                                                                                                                                                                                              |
| `orion-service-kafka-spring-boot-starter` | Spring Boot štartér pre službu                         | `orion-service-kafka-adapter`, `spring-boot-starter`                                                                                                                                                                                                                                                      |
