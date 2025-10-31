
---
title: "CloudStack Runtime Map — Core vs Plugin Modules"
date: 2025-10-31T12:45:00-04:00
draft: false
tags:
  - Apache CloudStack
  - Architecture
  - Plugins
  - Cloud Infrastructure
---

A concise runtime map of **Apache CloudStack** modules showing which are **core** vs **plugin**, and how they depend on each other.

---

## Runtime Layer Map (top → bottom)

```
──────────────────────────────────────────────
[ UI Layer ]  →  /ui/  (Vue.js frontend)
──────────────────────────────────────────────
      │  (calls REST APIs via /client/api)
      ▼
[ API Layer ]  →  /api/
──────────────────────────────────────────────
      │  Defines: BaseCmd, BaseAsyncCmd, annotations,
      │  response objects, validation, and command dispatch
      ▼
[ Service / Manager Layer ]  →  /server/, /engine/, /services/
──────────────────────────────────────────────
      │  Core business logic:
      │    - VmManager, NetworkManager, VolumeManager, etc.
      │  Uses DAOs, orchestrates async jobs and workflows
      ▼
[ Persistence Layer ]  →  /core/, /schema/
──────────────────────────────────────────────
      │  Defines:
      │    - Core entities (VMInstanceVO, NetworkVO, etc.)
      │    - Schema migrations
      ▼
[ Framework Layer ]  →  /framework/
──────────────────────────────────────────────
      │  Provides shared systems:
      │    - Plugin loader
      │    - Spring context injection
      │    - Event bus, config, job queue, scheduler
      ▼
[ Plugin Layer ]  →  /plugins/
──────────────────────────────────────────────
      │  Extends CloudStack with optional features:
      │    - Hypervisors (KVM, VMware)
      │    - Storage backends (Ceph, S3)
      │    - Networks (Nuage, Cisco)
      │    - Metrics (Prometheus)
      │    - Custom features (Coffee, AI, etc.)
      ▼
[ External Systems ]
──────────────────────────────────────────────
      Hypervisors, Storage Arrays, Network Controllers, etc.
```

---

## Relationships (Simplified Dependency Flow)

```
UI  →  API  →  Service  →  DAO  →  DB
            ↘  Plugin Managers ↙
             ↘ Framework (Events, Jobs)
```

---

## Core vs Plugin Summary

| Type     | Path(s)                                   | Description |
|-----------|-------------------------------------------|-------------|
| **Core** | `/api/`, `/server/`, `/engine/`, `/framework/`, `/core/`, `/schema/` | Defines the **stable backbone**. Handles API definitions, core services, async orchestration, and data persistence. |
| **Plugin** | `/plugins/` | Extends CloudStack without touching the core logic. Adds hypervisors, storage/network integrations, monitoring, and optional features. |

---

## Plugin Extensions

Plugins can inject new:

- **APIs** — `BaseCmd` / `BaseAsyncCmd` subclasses  
- **Services** — New `Manager` implementations  
- **Jobs** — Via Spring schedulers  
- **Integrations** — Network, storage, or metrics drivers  
