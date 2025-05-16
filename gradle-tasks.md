---
layout: default
title: Gradle úlohy a verzie
---

[<-- Na hlavnú stránku](index.md)

## Gradle úlohy

Každý modul v projekte používa štandardné Gradle úlohy, ako *clean*, *build* a *bootJar*. Tieto úlohy sú rovnaké pre všetky moduly.

- **clean**  
  Odstraňuje adresár `build` pre daný modul, čím zabezpečuje vyčistenie všetkých predchádzajúcich zostavení.

- **build**  
  Kompiluje zdrojový kód, spúšťa testy a vytvára výsledný artefakt. Zabezpečuje kompletné zostavenie modulu.

- **bootJar**  
  Vytvára spustiteľný JAR súbor pre Spring Boot aplikácie, ktorý obsahuje všetky potrebné závislosti. Výsledný súbor, ktorý sa nachádza v adresári `build/libs/`, je pripravený na nasadenie a spustenie pomocou príkazu `java -jar`.
### Moduly s názvom `*-spring-boot-starter`
Ak názov modulu končí na `*-spring-boot-starter`, pridáva sa doňho aj špeciálne nastavenie:
```groovy
tasks.named('compileJava') {
    inputs.files(tasks.named('processResources'))
}
```
Zabezpečuje, že pred kompiláciou Java kódu sa spracujú všetky zdroje, a teda:
  * Ak anotácie alebo kód závisia od obsahu súborov v `META-INF`, tieto budú už pripravené.
  * Pomáha správne rozpoznať a registrovať Spring Boot autokonfigurácie a komponenty pri zostavovaní.

## Ako zmeniť verzie závislostí v Gradle projekte

### Závislosti Spring

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
Verzie ostatných knižníc (napr. Kafka, Jackson, Lombok a pod.) môžete upraviť priamo v sekcii `dependencies` v príslušnom module:
```groovy
dependencies {
    implementation 'org.apache.kafka:kafka-clients:3.5.1'
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.15.3'
}
```

**Poznámka**: Po úprave verzií spustiťe príkaz `./gradlew clean build`.