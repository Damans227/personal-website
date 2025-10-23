---
title: "JVM Instrumentation for CloudStack Debugging"
date: 2025-10-22T15:33:37-05:00
draft: false
tags:
  - cloudstack
  - kvm
---

This guide provides a concise reference for using JVM diagnostic tools to monitor and debug the CloudStack Management Server, especially when installed via `.deb` packages.

---

## 1. Prerequisites

- CloudStack installed via `.deb` and running as a systemd service
- Java tools available (`jps`, `jstack`, `jmap`, `jstat`, `jcmd`)
- Access to `sudo`

Find CloudStack PID:
```bash
sudo jps -l
# or
ps aux | grep ServerDaemon
```

---

## 2. jps — List Java Processes

```bash
sudo jps -l
```

Shows running Java processes, including CloudStack's:
```
1191 org.apache.cloudstack.ServerDaemon
```

---

## 3. jstat — Monitor GC & Memory

```bash
sudo jstat -gc <PID> 1000
```

Shows garbage collection and memory stats every second.

Example:
```
S0C S1C S0U S1U EC EU OC OU MC MU CCSC CCSU YGC YGCT FGC FGCT GCT
```

Key:
- `EU`, `OU` = Eden/Old Gen Used
- `MU` = Metaspace Used
- `YGCT`, `FGCT` = GC times

---

## 4. jstack — Thread Dumps

```bash
sudo jstack <PID> > thread-dump.txt
```

Use to debug:
- UI/API hangs
- Deadlocks
- Stuck or blocked threads
- Excessive thread counts

Helpful filters:
```bash
grep -i "WAITING" thread-dump.txt
grep -A 10 "BLOCKED" thread-dump.txt
```

---

## 5. jmap — Heap Inspection

Check live heap:
```bash
sudo jmap -heap <PID>
```

Dump heap:
```bash
sudo jmap -dump:format=b,file=/tmp/cloudstack-heap.hprof <PID>
```

Analyze with Eclipse MAT or VisualVM.

---

## 6. jcmd — All-in-One Tool

List commands:
```bash
sudo jcmd <PID> help
```

Trigger GC:
```bash
sudo jcmd <PID> GC.run
```

Heap dump:
```bash
mkdir -p ./debugging
sudo jcmd <PID> GC.heap_dump ./debugging/cloudstack-heap.hprof
```
---

## Summary: Tool Usage Quick Table

| Tool    | Use Case                             |
|---------|--------------------------------------|
| `jps`   | Find running Java processes          |
| `jstat` | Monitor heap, GC, and memory         |
| `jstack`| Analyze thread dumps                 |
| `jmap`  | Inspect/dump heap                    |
| `jcmd`  | Trigger GC, dump heap, inspect flags |