---
layout: default
title: Kafka témy
---

[<-- Na hlavnú stránku](index.md)

### 1. instance-event
Do tejto témy jednotlivé mikroservisy automaticky posielajú informácie o svojom aktuálnom stave (napr. registrácia, "heartbeat", ukončenie činnosti).

| Názov poľa konfigurácie        | Hodnota   |
|----------------------|-----------|
| `CLEANUP_POLICY_CONFIG`     | `compact` |
| `MIN_COMPACTION_LAG_MS_CONFIG`     | `1`       |
| `MAX_COMPACTION_LAG_MS_CONFIG`   | `100`     |
| `MIN_CLEANABLE_DIRTY_RATIO_CONFIG`         | `0.001`   |
| `SEGMENT_MS_CONFIG`     | `10000`   |
| `DELETE_RETENTION_MS_CONFIG`    | `30000`   |

### 2. fetch-registry
Táto téma obsahuje aktuálne údaje centrálneho registra. Služba Orion ju napĺňa na základe informácií prijatých z témy `instance-event`.

| Názov poľa konfigurácie        | Hodnota   |
|----------------------|-----------|
| `CLEANUP_POLICY_CONFIG`     | `compact` |
| `MIN_COMPACTION_LAG_MS_CONFIG`     | `1`       |
| `MAX_COMPACTION_LAG_MS_CONFIG`   | `100`     |
| `MIN_CLEANABLE_DIRTY_RATIO_CONFIG`         | `0.001`   |
| `SEGMENT_MS_CONFIG`     | `10000`   |
| `DELETE_RETENTION_MS_CONFIG`    | `30000`   |

### 3. heartbeat-error
Do tejto témy posiela správy služba orions-service.
Tieto správy informujú o problémoch, ktoré nastali pri spracovaní požiadaviek pochádzajúcich z mikroservisov.
Všetky mikroservisy fungujúce ako objavovacie inštancie túto tému monitorujú (odoberajú), aby mohli automaticky riešiť problémy.
Okrem toho môže túto tému použiť administrátor systému na identifikáciu a riešenie problémov.

| Názov poľa konfigurácie        | Hodnota  |
|----------------------|----------|
| `CLEANUP_POLICY_CONFIG`     | `delete` |

### 4. node-event
Táto téma je veľmi podobná téme `instance-event`, ale obsahuje informácie o aktuálnom stave serverov Orion. Používa sa na výber lídra a monitorovanie stavu. *Poznámka: táto téma sa vyžaduje aj v jednoduchom režime.*

| Názov poľa konfigurácie        | Hodnota  |
|----------------------|----------|
| `CLEANUP_POLICY_CONFIG`     | `delete` |

### 5. data-replication
Táto téma obsahuje informácie podobné téme `fetch-registry`, ale podrobnejšie. Používa sa na replikáciu údajov medzi uzlami. *Poznámka: táto téma sa vyžaduje aj v jednoduchom režime.*

| Názov poľa konfigurácie        | Hodnota   |
|----------------------|-----------|
| `CLEANUP_POLICY_CONFIG`     | `compact` |
| `MIN_COMPACTION_LAG_MS_CONFIG`     | `1`       |
| `MAX_COMPACTION_LAG_MS_CONFIG`   | `100`     |
| `MIN_CLEANABLE_DIRTY_RATIO_CONFIG`         | `0.001`   |
| `SEGMENT_MS_CONFIG`     | `10000`   |
| `DELETE_RETENTION_MS_CONFIG`    | `30000`   |