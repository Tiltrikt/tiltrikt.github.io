---
layout: default
title: Služba objavovania Orion
description: Systémová príručka
---

## Table of Contents

- **Server**
    - [Getting Started](#getting-started)
    - [Configuration](#configuration)
    - [Exception Handling](#exception-handling)
    - [Contextual Data / Scoped Beans](#contextual-data--scoped-beans)
    - [Testing the Service](#testing-the-service)
    - [Server Events](#server-events)
    - [Security](#security)

- **Client**
    - [Getting Started](#getting-started-client)
    - [Configuration](#configuration-client)
    - [Security](#security-client)
    - [Tests with Grpc-Stubs](#tests-with-grpc-stubs)

- [Nasadenie](/deployment.md)
- [Trouble-Shooting](#trouble-shooting)
- [Example Projects](#example-projects)


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
