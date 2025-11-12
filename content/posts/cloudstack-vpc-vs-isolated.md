---
title: "CloudStack Networking: VPC vs Isolated Network"
date: 2025-11-12T13:40:00Z
draft: false
tags:
  - CloudStack
  - Networking
  - Virtualization
---

Both **VPCs** and **Isolated Networks** in Apache CloudStack provide Layer-3 network isolation for user VMs, but they differ in design and use cases.

---

## Isolated Network

An **Isolated Network** is a single, flat network with one virtual router.  
All VMs share the same subnet and routing domain.

**Key points:**
- One subnet per network (e.g., `10.1.1.0/24`).
- One virtual router handles NAT, DHCP, and firewall rules.
- No inter-tier routing; traffic is flat.
- Ideal for single-tier applications or quick deployments.

**Example:**
```
[VM1, VM2] ←→ Virtual Router ←→ Public IP
```

---

## Virtual Private Cloud (VPC)

A **VPC** contains multiple isolated networks (called **tiers**).  
Each tier has its own subnet and connects to a shared VPC virtual router.

**Key points:**
- Multi-tier design (web, app, DB).
- Centralized routing between tiers.
- Network ACLs control traffic between tiers and public networks.
- Supports Site-to-Site VPN, VPC-to-VPC VPN, private gateways, and user VPNs.

**Example:**
```
[Web Tier 10.0.1.0/24]
     ↓
[App Tier 10.0.2.0/24]
     ↓
[DB Tier 10.0.3.0/24]
```

---

## Comparison

| Feature | Isolated Network | VPC |
|----------|------------------|-----|
| **Scope** | One subnet | Multiple subnets (tiers) |
| **Router** | One per network | One per VPC |
| **Routing** | Flat, no inter-tier | Layer-3 routing between tiers |
| **ACLs** | Basic | Tier-based and advanced |
| **VPN Support** | Yes | Extended (Site-to-Site, VPC-to-VPC, User VPN) |
| **Private Gateway** | No | Yes |
| **Load Balancing** | Yes | Yes (inter-tier and inbound) |
| **Use case** | Simple or single-tier apps | Complex, multi-tier deployments |

---

## Summary

- **Isolated Network:** simple, one-tier network — fast to deploy, minimal control.  
- **VPC:** multi-tier environment with routing, ACLs, and VPN features — ideal for production and hybrid setups.

---