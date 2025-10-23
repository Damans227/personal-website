---
title: "CloudStack Maven Cheat Sheet"
date: 2025-10-22T15:33:37-05:00
draft: false
tags:
  - cloudstack
  - kvm
---

A quick cheat sheet for working with Apache CloudStack using Maven.

---

## Build Profiles

| Profile      | Description                                                  |
|--------------|--------------------------------------------------------------|
| `developer`  | Builds components useful during development (API, mgmt, etc).|
| `systemvm`   | Builds SystemVM templates and related artifacts.             |
| `noredist`   | Excludes non-distributable (e.g., proprietary) code.         |

---

## Common Maven Commands

### Full Build (excluding proprietary code):
```bash
mvn clean install -Pdeveloper -Dnoredist
```

### Full Build with SystemVM:
```bash
mvn clean install -Pdeveloper,systemvm -Dnoredist
```

### Build Specific Module:
```bash
mvn -Pdeveloper -pl <module-name> -am
```
- `-pl`: Only build this module
- `-am`: Also build required dependencies

**Example:**
```bash
mvn -Pdeveloper -pl server -am
```

---

##  Deploy the Database Schema

Used to initialize or reset the CloudStack database:
```bash
mvn -Pdeveloper -pl developer -Ddeploydb
```

---

## Running Jetty (for legacy UI testing)

Run Jetty for the CloudStack UI (usually used during frontend development):
```bash
mvn -pl client jetty:run
```

---

## Custom Java Options (`JAVA_OPTS`)

Useful for overriding JVM behavior (heap size, GC logging, IPv4 stack, etc.)

Example:
```bash
export JAVA_OPTS="-Xmx4G -Djava.net.preferIPv4Stack=true"
```

Applied via systemd override or environment file for `cloudstack-management`.

---

## Build affected modules:

   ```bash
   mvn -Pdeveloper -pl server -am
   ```
---

## Final Packaging (DEB/RPM/TAR)

```bash
mvn clean package -Pdeveloper,systemvm -Dnoredist
```

Creates `.deb`, `.rpm`, `.tar.gz` in `dist/target/`.

---

## Key Modules

| Module             | Description                        |
|--------------------|------------------------------------|
| `api`              | API interfaces and command defs    |
| `server`           | Core logic, services, managers     |
| `client`           | UI code (old UI)                   |
| `plugins/*`        | Optional plugins (e.g. hypervisors)|
| `developer`        | Dev tools (e.g., db deployment)    |
| `utils`            | Shared utility classes             |
