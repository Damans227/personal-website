---
title: "CloudStack Logging Guide"
date: 2025-10-22T15:33:37-05:00
draft: false
tags:
  - cloudstack
  - kvm
---

# CloudStack Logging Guide

## 1. Where CloudStack Logs Live

CloudStack has three main components — each with its own logs:

| Component          | Purpose                          | Log Path                                           |
|--------------------|----------------------------------|----------------------------------------------------|
| Management Server  | UI, API, core business logic     | `/var/log/cloudstack/management/management-server.log` |
| KVM Agent (if used)| Talks to libvirt/hypervisor      | `/var/log/cloudstack/agent/agent.log`             |
| Usage Server       | Tracks usage for billing/stats   | `/var/log/cloudstack/usage/usage.log`             |


Worked a lot with:

`/var/log/cloudstack/management/management-server.log`

---

## 2. How to Explore Logs Like a Pro

**Read logs live (tailing)**

```bash
tail -f /var/log/cloudstack/management/management-server.log
```

**Use grep to filter:**

```bash
tail -f management-server.log | grep -i error
```

**Or generate fresh new logs (rotated):**

```bash
ls -lh /var/log/cloudstack/management/
less /var/log/cloudstack/management/management-server.log.1
```

---

## 3. Understanding Log Messages

CloudStack logs are written by Java code using log4j format.

**Typical line:**
```
2025-09-27 01:07:10,016 WARN  [o.a.c.s.l.r.RegistryLifecycle] (main:[]) (logid:) Bean registration failed for ...
```

**Breakdown:**

| Field                        | Meaning                                          |
|-----------------------------|--------------------------------------------------|
| 2025-09-27 01:07:10,016      | Timestamp                                        |
| WARN                        | Log level (DEBUG, INFO, WARN, ERROR, FATAL)     |
| [o.a.c.s...]                | Logger (package name; Java-style)               |
| main                        | Thread name                                     |
| (logid:)                    | Optional request ID (empty unless API call)     |
| Message                     | Human-readable description of the event         |

---

## 4. Configuring Logging (log4j)

**Config file:**

`/etc/cloudstack/management/log4j-cloud.xml`

**Key settings:**

```xml
log4j.rootLogger=INFO, file
```

You can change `INFO` to `DEBUG` for more detail (temporarily)

**To apply changes:**

```bash
sudo systemctl restart cloudstack-management
```

---

## 5. Use Cases for Logs

### Startup Issues

```bash
grep -i jetty /var/log/cloudstack/management/management-server.log
```

Look for `Jetty started`, `8080`, or port binding issues.

---

### Auth/API Issues

```bash
grep -i auth /var/log/cloudstack/management/management-server.log
```

You might see:

```
Unable to verify user credentials and/or request signature
```

---

### UI/API Unreachable

```bash
grep -i exception /var/log/cloudstack/management/management-server.log
```

Or:

```bash
ss -tuln | grep 8080
```

---

### Resource Errors (e.g. deploying VMs)

```bash
tail -f management-server.log | grep -i vm
```