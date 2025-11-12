---
title: "CloudStack Snapshots: VM vs Volume"
date: 2025-11-12T13:30:00Z
draft: false
tags:
  - CloudStack
  - Storage
  - Virtualization
---

Snapshots in Apache CloudStack can exist at two levels — **VM snapshots** and **volume snapshots** — each serving different purposes and stored in different locations.

---

## VM Snapshots

A **VM snapshot** is a hypervisor-level checkpoint of an entire virtual machine.  
It captures the VM’s disk state and optionally its memory, allowing fast rollback to a previous state.

**Key points:**
- Stored on **primary storage**.
- Created quickly without full data copy.
- Can include **memory state** for live restore.
- Used mainly for **temporary rollback** or testing.
- **Cannot** be downloaded or used to create templates.

**Example use:** Before applying OS patches or configuration changes.

---

## Volume Snapshots

A **volume snapshot** is a backup of a single disk — root or data — taken at the storage layer.  
It’s stored in **secondary storage** and is suitable for long-term backups or template creation.

**Key points:**
- One snapshot per volume.
- Can be **manual** or **scheduled**.
- Can be **restored**, **cloned**, or **converted** to a template.
- Safe from primary storage failure.
- Slower to create but persistent.

**Example use:** Backing up a database volume or creating a reusable OS template.

---

## Comparison

| Aspect | VM Snapshot | Volume Snapshot |
|--------|--------------|----------------|
| **Scope** | Entire VM (optionally memory) | Single volume |
| **Storage** | Primary storage | Secondary storage |
| **Use case** | Quick rollback | Long-term backup |
| **Template creation** | No | Yes (root volume only) |
| **Speed** | Fast | Slower |
| **Persistence** | Temporary | Durable |

---

## Summary

- **VM snapshots**: short-term restore points on primary storage.  
- **Volume snapshots**: long-term backups on secondary storage.  
- Use VM snapshots for quick reverts and volume snapshots for data protection or template creation.

---
