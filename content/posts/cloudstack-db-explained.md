---
title: "CloudStack Databases Explained: Foundation of the Management Server"
date: 2025-11-04T10:00:00Z
draft: false
tags:
  - CloudStack
  - Database
  - MySQL
  - Infrastructure
---

CloudStack relies on two key databases — **cloud** and **cloud_usage**.

---

## Overview

The CloudStack Management Server interacts continuously with a database backend. Every API call, VM operation, or event updates these databases.

```
cloudstack-management
        │
        ▼
   MySQL Server
     ├── cloud
     └── cloud_usage
```

---

## Database Roles

### `cloud`

The **core operational database**.  
Stores all configuration and runtime state.

**Key responsibilities:**
- VM instances, volumes, and templates  
- Networking, IP addresses, and VLANs  
- User accounts, roles, and API keys  
- Async jobs, events, and orchestration logs

---

### `cloud_usage`

The **usage and metering database**.  
Stores data for reporting, billing, and analytics.

**Key responsibilities:**
- Resource usage (CPU, memory, network)  
- Event-driven usage tracking  
- Historical data for metering systems

---

## MySQL Configuration for CloudStack

Before deployment, MySQL must be tuned for high concurrency and transactional safety.

Example configuration (`/etc/mysql/mysql.conf.d/mysqld.cnf`):

```ini
[mysqld]
server_id = 1
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=1000
log-bin=mysql-bin
binlog-format='ROW'
```

**Why it matters:**
- Prevents silent data corruption  
- Allows long-running transactions  
- Ensures durable writes  
- Supports future replication setups

---

## Initialization

CloudStack includes a setup utility to create and seed both databases:

```bash
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root: -i 192.168.122.10
```

This command:
1. Connects to MySQL as `root`
2. Creates `cloud` and `cloud_usage`
3. Creates the user `cloud@localhost` with password `cloud`
4. Populates the schema and default data
