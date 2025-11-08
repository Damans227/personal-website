---
title: "CloudStack Primary Storage: Local vs Shared"
date: 2025-11-08T14:25:00Z
draft: false
tags:
  - CloudStack
  - Storage
  - Infrastructure
---

Primary storage in CloudStack holds the **root volumes** of running virtual machines and **data volumes** attached to them. It operates at the **cluster level** — all hosts within a cluster share access to the same primary storage pool.

---

## Local Storage

Each hypervisor uses its own local disk for VM volumes. There is no shared storage between hosts.

### Characteristics
- No need for expensive centralized SAN/NAS.
- Limited functionality — no live migration or HA.
- If a host fails, all VMs on it are lost.
- Suitable for **stateless workloads** (temporary compute nodes).

### CloudStack Behavior
- Each host registers its local storage automatically.
- You enable this during zone setup by checking *“Use local storage for VMs.”*

---

## Shared Storage

All hosts in the cluster connect to a **common storage pool** over the network.

### Characteristics
- Enables **live migration** and **high availability (HA)**.
- Supports failover — VMs can move between hosts without downtime.
- Typically more expensive but provides enterprise reliability.

### Supported Storage Types
- NFS (Network File System)
- iSCSI
- Fibre Channel
- OCFS2 / CLVM (advanced setups)

CloudStack can manage these directly or attach to pre-configured storage pools.

---

## Comparison

| Type | Access | Live Migration | HA | Use Case |
|------|---------|----------------|----|-----------|
| **Local** | Per-host | ❌ No | ❌ No | Cheap, stateless workloads |
| **Shared** | Cluster-wide | ✅ Yes | ✅ Yes | Production, HA, migrations |

---

## Key Takeaways

- **Local storage** → simple, cheap, isolated per host, no HA.  
- **Shared storage** → cluster-wide, required for live migration and failover.  
- Choice depends on cost, workload type, and desired fault tolerance.

---