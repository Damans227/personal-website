---
title: "CloudStack Architecture: The Hierarchy Explained"
date: 2025-11-03T10:00:00Z
draft: false
tags:
  - CloudStack
  - Architecture
  - Infrastructure
---

CloudStack organizes infrastructure hierarchically. Each level serves a specific purpose.

## The Hierarchy

```
Region (Geographically dispersed)
  └─ Zone (Usually = 1 datacenter)
      └─ Pod (Usually = 1 physical rack)
          └─ Cluster (Same hypervisor type)
              └─ Host (Individual physical computer)
```

## Regions

Collection of zones across different locations. If one datacenter fails, other regions keep running.

**Visible to users:** Yes

## Zones

Typically one datacenter. Provides physical isolation (separate power, network). Contains pods and secondary storage.

**Visible to users:** Yes

## Pods

Usually one physical rack. Contains clusters and primary storage. All hosts in a pod are on the same subnet.

**Visible to users:** No

## Clusters

Identical hosts running the same hypervisor. Enables live migration between hosts without downtime.

**Requirements:**
- Same hypervisor (all KVM, or all Hyper-V, etc.)
- Identical hardware
- Same subnet
- Shared Primary Storage

**Visible to users:** No

## Hosts

Individual physical computers with hypervisor software. Run the instances.

**Visible to users:** No

---

## Storage

**Primary Storage:** Cluster/zone-level. Stores virtual disks of running instances.

**Secondary Storage:** Zone-wide. Stores templates, ISOs, snapshots.

---

## Networking

**Basic:** Single network per pod. Layer-3 isolation (firewall rules). Not for multi-tenant.

**Advanced:** Multiple networks per zone. Layer-2 isolation (VLANs). Suitable for multi-tenant.

**Traffic types:**
- Guest: Instance-to-instance
- Management: CloudStack system communication
- Storage: Template/snapshot copying
- Public (advanced only): Internet access

---

## Why This Matters

Each level isolates failures:
- Region failure: Other regions survive
- Zone failure: Other zones survive
- Pod failure: Other pods survive
- Cluster failure: Other clusters survive

Homogeneous clusters enable live migration (zero-downtime maintenance).

Different hypervisors in different clusters gives flexibility.