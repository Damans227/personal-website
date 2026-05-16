---
title: "AWS EC2: Instance Storage"
date: 2026-05-16T16:00:00Z
draft: false
tags:
  - AWS
  - EC2
  - Storage
---

Once an EC2 instance is running, it needs somewhere to put data. AWS gives you three kinds of disk to attach to a VM, and each one behaves very differently.

---

## The options

| Type | Storage | Lifecycle | Scope |
|------|---------|-----------|-------|
| **EBS** | Block, network-attached | Persistent | One AZ |
| **Instance Store** | Block, physically on the host | Ephemeral — lost on stop/terminate | One host |
| **EFS** | File (NFS), network share | Persistent | Multi-AZ |

---

## Mental model

```
EC2 ──attaches──> EBS volume     (its own dedicated disk, persistent)
EC2 ──has──────> Instance Store  (host's physical NVMe, ephemeral)
EC2 ──mounts───> EFS             (network file share, many EC2s share it)
```

In plain terms:

- **EBS** is your own hard drive — one EC2, persistent.
- **Instance Store** is a scratch SSD on the host — fast, but dies with the host.
- **EFS** is a shared network drive — many EC2s, multi-AZ.

---

## EBS — the workhorse

**What it is:** network-attached block storage. To the OS it looks like an ordinary disk.

**Key facts:**

- **AZ-scoped** — the volume and the EC2 instance must be in the same Availability Zone.
- **Persistent** — survives an instance stop or terminate, unless it is set to delete on termination.
- One volume attaches to one instance by default. **Multi-Attach** exists for `io1`/`io2`, but it is niche.
- You pay for the **provisioned size**, whether the volume is full or empty.

**Volume types:**

- **gp3 / gp2** — general purpose SSD. The default choice.
- **io1 / io2** — high-IOPS SSD, for databases.
- **st1** — throughput HDD, for big sequential reads.
- **sc1** — cold HDD, the cheapest option, for rarely accessed data.

**Snapshots:**

- A point-in-time backup, stored in S3 (managed by AWS).
- **Region-scoped** — unlike volumes, which are AZ-scoped.
- **Incremental** — only changed blocks are saved.
- Used for backup, moving a volume across AZs, cloning, cross-region disaster recovery, and building AMIs.

---

## Instance Store — fast and ephemeral

- Physical NVMe sitting on the host machine itself.
- Very fast — the lowest latency and highest IOPS available.
- **Lost on stop, terminate, or host failure.**
- Comes free with certain instance types (`i3`, `m5d`, and others).
- Use it for cache, scratch space, temp data, or replicated databases that handle their own durability.

---

## EFS — shared file system

- A managed NFS file system, mountable by **many EC2s at once**.
- **Multi-AZ** by default.
- Auto-scales capacity — you pay per GB used, not per GB provisioned.
- Linux only (NFSv4). For Windows, use **FSx for Windows** instead.
- Use it for shared web content, home directories, or shared ML datasets.

---

## Running example

A **photo app** running on EC2 needs three different kinds of storage:

1. **Root disk** (OS and app code) → **EBS gp3** — persistent, AZ-scoped.
2. **Scratch space for image processing** → **Instance Store** — fast, and fine to lose.
3. **Shared assets across all EC2s** (a user avatars cache, say) → **EFS** — multi-AZ and shared.

The user photos themselves go to **S3**, not to any of these. EBS, Instance Store, and EFS are for *running the app*; S3 is for *durable user data*.

---

## EBS vs EFS — the classic question

| | EBS | EFS |
|---|-----|-----|
| Storage type | Block (looks like a disk) | File (NFS share) |
| Attached to | 1 EC2 (usually) | Many EC2s |
| Scope | One AZ | Multi-AZ |
| You format it? | Yes (`mkfs`) | No — it is already a filesystem |
| Cost model | Provisioned size | Pay per GB used |

---

## Key principles

- Every EC2 needs a **root EBS volume** — it is effectively required.
- **Snapshot regularly** — use Data Lifecycle Manager (DLM) to automate it.
- **Instance Store is scratch** — never put data you can't afford to lose on it.
- Reach for **EFS to share, EBS for a dedicated disk, and Instance Store for speed and scratch.**
- Volumes are **AZ-locked**, but snapshots are **region-wide** — that is how you move a volume between AZs.

---

## Summary

- **EBS** is a dedicated disk — persistent, scoped to one AZ.
- **Instance Store** is scratch space — fast, but it dies with the host.
- **EFS** is a shared network drive — persistent and multi-AZ.
- These three are for running the app; durable user data belongs in S3.

---
