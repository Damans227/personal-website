---
title: "Debugging CloudStack UI Access Failure"
date: 2025-10-22T15:33:37-05:00
draft: false
tags:
  - cloudstack
  - kvm
---

## The Problem

Cloudstack UI was not loading. The browser showed “connection refused”.  
So I tried the same from the **Linux host**:

```bash
curl http://192.168.122.10:8080/client
```

Result:

```
curl: (7) Failed to connect
```

This confirmed that the **problem wasn't the tunnel or Mac**, but that **the Linux host couldn’t reach the CloudStack VM either**.

---

## Diagnostics

### Step 1: Is the management service running?

```bash
sudo systemctl status cloudstack-management
```

It was running fine.

---

### Step 2: Is anything listening on port 8080?

```bash
sudo ss -tuln | grep 8080
```

No output.

```bash
sudo lsof -i :8080
```

Output:

```
java  18900  cloud  38u  IPv6  ...  TCP *:http-alt (LISTEN)
```

Something was listening — but **not via IPv4**.

---

### Step 3: Confirm IPv6 binding

```bash
sudo ss -tuln6 | grep 8080
```

Output:

```
tcp   LISTEN  0  50  *:8080  *:*
```

This showed that Jetty was bound to `::` (IPv6 all interfaces), but **not to 0.0.0.0 (IPv4)**.

---

## Root Cause

Jetty (CloudStack’s embedded server) was binding only to IPv6, and the Linux host (and Mac via tunnel) was trying to connect via IPv4.

On modern Linux + JVM, binding to `::` **does not automatically include IPv4** unless `IPV6_V6ONLY=0`, which is not the default.

---

## The Fix — Forcing IPv4 Binding

### Step 1: Update Jetty binding

Edit CloudStack’s config file:

```bash
sudo nano /etc/cloudstack/management/server.properties
```

Add this line:

```properties
host=0.0.0.0
```

This tells Jetty to bind explicitly to all **IPv4 interfaces**.

---

### Step 2: Force JVM to prefer IPv4 stack

Create a systemd service override:

```bash
sudo mkdir -p /etc/systemd/system/cloudstack-management.service.d
sudo nano /etc/systemd/system/cloudstack-management.service.d/override.conf
```

Paste:

```ini
[Service]
Environment="JAVA_OPTS=-Djava.net.preferIPv4Stack=true"
```

---

### Step 3: Reload systemd and restart CloudStack

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart cloudstack-management
```

---

### Step 4: Confirm success

Check if CloudStack is now listening on IPv4:

```bash
sudo ss -tuln | grep 8080
```

Got the correct output:

```
tcp   LISTEN  0  50  0.0.0.0:8080  0.0.0.0:*
```

Test from Linux host:

```bash
curl http://192.168.122.10:8080/client
```

Received HTML response — success!


## Summary of Fix

| What Broke                 | Why It Happened                   | How I Fixed It                                        |
|----------------------------|-----------------------------------|--------------------------------------------------------|
| UI not reachable on port 8080 | Jetty bound to IPv6 only         | Added `host=0.0.0.0` in `server.properties`            |
| No output from `ss -tuln`  | Java socket was `IPV6_V6ONLY=1`   | Added JVM flag `preferIPv4Stack=true`                 |
| Tunnel from Mac didn’t work | IPv4 traffic never reached Jetty | Bound Jetty to IPv4 + forced JVM to prefer it         |
| Service looked “healthy”   | Jetty started silently on IPv6    | Diagnosed with logs, `lsof`, and `ss`                 |

---

## 🔗 References

- 📘 [Apache CloudStack Documentation](https://docs.cloudstack.apache.org/)
- 🚀 [Jetty Server Configuration Docs](https://www.eclipse.org/jetty/documentation/)
- ☕ [Java Networking Properties](https://docs.oracle.com/javase/8/docs/api/java/net/doc-files/net-properties.html)