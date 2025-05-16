---
layout: default
title: Začiatok práce s klientom
---

[<-- Na hlavnú stránku](index.md)

### Spôsob použitia
Do mikroslužby je potrebné pripojiť nasledujúce závislosti:
```groovy
dependencies {
    implementation "org.springframework.boot:spring-boot-starter-web"
    implementation "dev.tiltrikt:orion-client-kafka-spring-boot-starter:1.0.1"
}
```

**Poznámka:** *aby ste mohli úspešne stiahnuť a použiť závislosť, musíte pridať Maven repozitár GitHub Packages do bloku `dependencyResolutionManagement` vo vašom Gradle projekte.*
```groovy
dependencyResolutionManagement {
    repositories {
        mavenCentral()
        mavenLocal()
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/tiltrikt/orion")
            credentials {
                username = <YOUR_USERNAME>
                password = <YOUR_PRIVATE_ACCESS_TOKEN>
            }
        }
    }
}
```

### Príklad konfigurácie klienta objavovacej služby
**Klient vyhľadávacej služby** - inštancia, ktora používa údaje registra.
```yaml
server:
  port: 8083

spring:
  application:
    name: fetch-registry-client

orion:
  client:
    fetch-registry: true
    self-registration: false
```

### Príklad konfigurácie objavovacej inštancie
**Objavovacia inštancia** - inštancia, ktorá aktívne odosiela informácie o svojom stave, aby mohla byť objavená a monitorovaná.
```yaml
server:
  port: 8084

spring:
  application:
    name: self-registration-client

orion:
  client:
    fetch-registry: false
    self-registration: true
    heartbeat-rate-sec: 5
    lease-duration-sec: 15
```

### Zoznam všetkých parametrov pre konfiguráciu

| Názov                                  | Popis                                                                                                                                                         |
|----------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `orion.client.fetch-registry`          | Určuje, či má klient Orion automaticky načítať informácie registra pri štarte.                                                                                |
| `orion.client.heartbeat-rate-sec`      | Interval v sekundách, v ktorom klient posiela správy "srdcový tep" serveru Orion. Nesmie byť kratší ako `lease-duration-sec`.                                 |
| `orion.client.kafka.bootstrap-servers` | Adresa Kafka brokera/brokerov, ku ktorým sa klient Orion pripája pre publikovanie a odoberanie správ.                                                         |
| `orion.client.kafka.trusted-packages`  | Zoznam názvov balíkov oddelených čiarkou, ktoré Kafka klient považuje za dôveryhodné pre deserializáciu.                                                      |
| `orion.client.lease-duration-sec`      | Trvanie v sekundách, počas ktorého sa služba považuje za dostupnú od posledného prijatého srdcový tepu. Musí byť väčšie ako `orion.client.heartbeat-rate-sec`. |
| `orion.client.self-registration`       | Určuje, či sa má klient Orion automaticky zaregistrovať na serveri Orion pri štarte.                                                                          |
| `orion.client.kafka.group-id`          | Každý klient musí mať rozdielne group ID, aby získal celý register.                                                                                           |