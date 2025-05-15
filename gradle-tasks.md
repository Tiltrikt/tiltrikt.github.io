---
layout: default
title: Gradle úlohy
---

[<-- Na hlavnú stránku](index.md)

* `build` 

* `clean`
* `bootjar`

## Ako zmeniť verzie závislostí v Gradle projekte

### Hlavné závislosti Spring

Ak chcete zmeniť verzie Spring Boot alebo súvisiacich pluginov, prejdite do **hlavného súboru `build.gradle`** nachádzajúceho sa v koreňovom adresári projektu. V sekcii `plugins` môžete vidieť niečo podobné:

```groovy
plugins {
    id 'idea'
    id 'java-library'
    id 'org.springframework.boot' version '3.4.1'
    id 'io.spring.dependency-management' version '1.1.7'
    id 'maven-publish'
}
```
Tu môžete upraviť verzie napríklad:

  * `org.springframework.boot` — verzia Spring Boot
  * `io.spring.dependency-management` — Spring Dependency Management plugin

### Ostatné závislosti
Verzie ostatných knižníc (napr. Kafka, Jackson, Lombok a pod.) môžete upraviť priamo v sekcii dependencies v príslušnom module:
```groovy
dependencies {
    implementation 'org.apache.kafka:kafka-clients:3.5.1'
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.15.3'
}
```

**Poznámka**: Po úprave verzií spustiťe príkaz `./gradlew clean build`.