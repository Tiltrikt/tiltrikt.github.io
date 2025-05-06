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
