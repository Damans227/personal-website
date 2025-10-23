---
title: "CloudStack Virtual Router Overview"
date: 2025-10-22T15:33:37-05:00
draft: false
tags:
  - cloudstack
  - kvm
---

# CloudStack Virtual Router Overview

## In CloudStack:

**Isolated Networks** and **VPCs** depend on a **Virtual Router (VR)** for all **external (north-south) connectivity**.

Without a VR:

- VMs can still talk to each other within the same network (L2/L3 via bridge)
- But they **cannot reach outside**, and **nothing external can reach them** (no NAT, no DHCP, no gateway, no DNS)

---

## Why?

Because in both **isolated networks** and **VPCs**, the **VR acts as**:

- Default gateway for VMs  
- DHCP server (VMs won’t even get IPs without it)  
- DNS forwarder  
- NAT/firewall/VPN endpoint  
- Router for public traffic  

---

##  Network Flow Diagram

```
                Public Network (203.x.x.x)
                          |
                          v
        +-------------------------------------+
        |      Virtual Router (VR)           |
        |-------------------------------------|
        | NIC1: 203.x.x.5 (Public IP)         |
        | NIC2: 10.1.1.1 (Guest Network GW)   |
        +----------------+--------------------+
                          |
                          v
             Guest Isolated Network (10.1.1.0/24)
                          |
        +-----------------+-------------------+
        |                                     |
        v                                     v
   Guest VM 1                           Guest VM 2
  IP: 10.1.1.10                          IP: 10.1.1.11
```