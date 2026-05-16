---
title: "AWS EC2: Elastic Compute Cloud"
date: 2026-05-16T14:00:00Z
draft: false
tags:
  - AWS
  - EC2
  - Compute
---

EC2 is the **compute layer** of AWS — rentable virtual machines. When you need a server to run an application, EC2 is where it lives.

---

## The core pieces

Launching an EC2 instance means assembling a handful of parts. Each one answers a specific question.

| Piece | What it is |
|--------|------------|
| **Instance** | A running VM |
| **AMI** | The image/template the VM boots from — OS plus pre-installed software |
| **Instance Type** | Sizing — CPU, RAM, network (e.g. `t3.medium`, `m5.large`) |
| **Key Pair** | The SSH key used to log in |
| **Security Group** | A stateful firewall controlling inbound and outbound traffic |
| **User Data** | A script that runs on first boot to bootstrap the instance |

---

## Mental model

```
AMI (template) + Instance Type (size) ──launch──> Instance (running VM)
                                                       │
                                                       ├── Security Group (firewall)
                                                       ├── Key Pair (login)
                                                       ├── IAM Role (AWS permissions)
                                                       └── Storage (EBS / Instance Store)
```

An instance is the AMI and instance type combined, with a firewall, a login key, AWS permissions, and storage hanging off it.

---

## How it works — a running example

Suppose you want to launch a **web server**:

1. **Pick an AMI** → Amazon Linux 2023 (the OS and base packages).
2. **Pick an instance type** → `t3.medium` (2 vCPU, 4 GB RAM).
3. **Attach a key pair** → so you can SSH in.
4. **Attach a security group** → allow port 22 (SSH from your IP) and port 443 (HTTPS from anywhere).
5. **Attach an IAM role** → `PhotoAppRole`, so the app can reach S3 without embedded keys.
6. **Add a user data script** → installs Nginx and pulls the app code on first boot.
7. **Click launch** → AWS picks a host, provisions the VM, and boots it from the AMI.

The app is now live. You can SSH in with the key pair, or let the user data script do everything for you.

---

## Instance lifecycle

An instance moves through a few states, and each one has cost implications:

- **Running** → billed.
- **Stopped** → not billed for compute, but the EBS volumes are still charged. Can be restarted.
- **Terminated** → gone forever. The EBS root volume is usually deleted with it.
- **Hibernate** → saves RAM to EBS for a faster restart.

The key takeaway: **stopped is not the same as terminated** — a stopped instance still costs money for its storage.

---

## Instance type families

Instance type names follow a simple pattern: the **letter** is the workload shape, the **number** is the generation, and the **suffix** is a variant (`g` = ARM, `n` = network, `d` = local disk).

- **T** (`t3`, `t4g`) → burstable, cheap, general purpose.
- **M** (`m5`, `m6i`) → balanced general purpose.
- **C** (`c5`, `c6i`) → compute-optimized, for CPU-heavy work.
- **R** (`r5`, `r6i`) → memory-optimized, for RAM-heavy workloads like databases.
- **I / D** → storage-optimized, with local NVMe.
- **G / P** → GPU, for ML and graphics.

---

## Purchasing options

EC2 offers several cost models. The right one depends on how predictable and how interruptible your workload is.

| Type | When to use it |
|------|----------------|
| **On-Demand** | Pay per hour/second, no commitment. The default. |
| **Reserved (RI)** | A 1- or 3-year commitment for a big discount. Steady workloads. |
| **Savings Plan** | A flexible commitment on $/hour spend. |
| **Spot** | Up to 90% off, but AWS can reclaim the instance with a 2-minute notice. Fault-tolerant workloads. |
| **Dedicated Host** | A physical server reserved entirely for you. Licensing and compliance needs. |

---

## Security Groups — the important details

Security groups are the instance-level firewall, and a few of their properties are worth calling out:

- **Stateful** → if you allow traffic inbound, the response is automatically allowed back out.
- **Allow rules only** → there are no deny rules. Denies happen at the subnet level via NACLs.
- **Can reference other security groups**, not just IP ranges.
- **Multiple security groups** can attach to one instance — their rules are additive.

---

## Key principles

- Use **IAM roles**, not embedded credentials.
- Treat **security groups** as the instance-level firewall.
- Use **user data** as the bootstrap script for fresh instances.
- Bake your own **AMIs** — a golden image gives fast, reproducible launches.
- Remember **stopped ≠ terminated** — stopped instances still cost money for their EBS volumes.

---

## Summary

- EC2 provides rentable virtual machines — the compute layer of AWS.
- The one-line essence: **EC2 = AMI + instance type + storage + security group + IAM role = a running VM.**
- Instance type names encode the workload shape, generation, and variant.
- Purchasing options trade commitment and interruptibility for cost.
- Watch the lifecycle: running and stopped instances both incur charges, only terminated ones stop the bill (and take their storage with them).

---
