---
title: "CloudStack Communication Ports Overview"
date: 2025-11-10T14:45:00Z
draft: false
tags:
  - CloudStack
  - Networking
  - Ports
  - CPVM
  - SSVM
---

CloudStack components communicate across multiple networks and ports.

---

## Port Summary Table

| Source / Target | Port(s) | Purpose / Description |
|-----------------|----------|------------------------|
| **User → Management Server** | 8080 / 8096 | CloudStack UI / API |
| **Management Server ↔ Management Server** | 9090 / 8250 | Clustered management coordination |
| **Management Server ↔ MySQL** | 3306 | Database connection |
| **CPVM ↔ Management Server** | 8250 | Console proxy and control communication |
| **SSVM ↔ Management Server** | 8250 | Secondary storage operations (template, ISO, snapshot jobs) |
| **Virtual Router ↔ Management Server** | 3922 | SSH control and configuration |
| **SSVM ↔ Secondary Storage (NFS)** | 111 / 2049 | NFS mount and data transfer |
| **CPVM ↔ Hypervisors** | 22 / 443 | Console proxy, authentication, and HTTPS access |
| **SSVM ↔ HTTP File Share** | 80 / 443 | Template and ISO downloads |
| **User Browser ↔ CPVM** | 443 / 80 | HTTPS console access for VM consoles |
| **Management Server ↔ Xen Hosts** | 22 / 80 / 443 | Agent management, API communication |
| **Management Server ↔ KVM Hosts** | 22 | Agent setup via SSH |
| **Management Server ↔ vCenter (ESXi)** | 443 | vCenter API communication |
| **Virtual Router ↔ Secondary Storage** | 111 / 2049 | Template and snapshot copy operations |

---

## Accessing System VMs (CPVM / SSVM / VR)

CloudStack deploys system VMs (such as **CPVM**, **SSVM**, and **Virtual Routers**) with an **isolated link-local IP** and restricted SSH access.  
You can connect to these from the **Management Server** using the CloudStack SSH key located at `/root/.ssh/id_rsa.cloud`.

Example:

```bash
root@cs-user-lab:~# ssh -i /root/.ssh/id_rsa.cloud -p 3922 root@169.254.221.107
```

This connects to the **System VM (CPVM, SSVM, or VR)** via the **link-local network** using the **3922 port**.

---
