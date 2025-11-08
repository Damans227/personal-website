---
title: "CloudStack Primary Storage: How Linked Clones Work"
date: 2025-11-08T14:45:00Z
draft: false
tags:
  - CloudStack
  - Storage
  - Virtualization
---

When CloudStack deploys a VM from a template, it uses a **linked clone** instead of creating a full copy of the disk. This approach saves both time and storage space.

---

## What Is a Linked Clone

A **linked clone** is a virtual disk that shares the same **base image** as another VM or template.  
It doesn’t duplicate the full data. Instead:

- The **base disk** (template) is read-only.  
- The **linked clone** stores only changes (writes made by the VM).

Every read operation for unchanged data is fetched from the base disk, while every write creates a new block in the clone’s own delta file.

---

## How It Works

Example with KVM and qcow2 disks:

```
template.qcow2   → base image (read-only)
vm1.qcow2        → linked clone (delta file)
```

Command example:

```bash
qemu-img create -f qcow2 -b template.qcow2 vm1.qcow2
```

When `vm1` runs:
- Unchanged data is read from `template.qcow2`.
- Modifications are written into `vm1.qcow2`.

---

## Benefits

| Benefit | Explanation |
|----------|--------------|
| **Space-efficient** | Only stores VM-specific changes. |
| **Fast deployment** | No full disk copy required. |
| **Template consistency** | All clones share a common base image. |

---

## Drawbacks

| Drawback | Explanation |
|-----------|-------------|
| **Single point of failure** | If the base image is lost or corrupted, all clones fail. |
| **Performance degradation** | Long clone/snapshot chains slow down disk I/O. |
| **Needs consolidation** | Over time, linked clones may need to be merged into full disks. |

---

## How CloudStack Uses It

When deploying a VM:

1. CloudStack copies the **template** from **secondary** to **primary storage** (once per cluster).
2. The **hypervisor** (KVM, XenServer, or VMware) creates a **linked clone** of that base disk.
3. Each VM’s disk references the same base image but keeps its own delta file.
4. CloudStack tracks these relationships internally for volume management.

---

## Key Takeaways

- A **linked clone** is a delta-based copy that references a base image.  
- It’s efficient for space and deployment speed but depends on the parent image’s integrity.  
- CloudStack leverages linked clones to optimize VM provisioning on primary storage.

---