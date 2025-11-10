---
title: "Understanding CloudStack Hypervisors — KVM vs XenServer vs VMware"
date: 2025-11-10T12:30:00-05:00
draft: false
tags:
  - cloudstack
  - kvm
  - xenserver
  - vmware
---

CloudStack supports three main hypervisors — **KVM**, **XenServer**, and **VMware (vSphere/ESXi)**. Each integrates differently with the CloudStack management plane.

## Overview

| Feature | **KVM** | **XenServer** | **VMware (vSphere)** |
|----------|----------|----------------|----------------------|
| CloudStack connects to | Each host directly | Pool Master | vCenter API |
| System VM control | Link Local network | Link Local network | Management network |
| Networking | Linux bridge / OVS | Network labels | vSwitch / dvSwitch |
| Storage | NFS / SharedMountPoint | NFS / iSCSI / FC | vCenter-managed datastores |
| Max hosts/cluster | Unlimited | 64 (v7.5) | 32 (v5.5), 64 (v6.0+) |
| HA | Yes (via libvirt) | Yes (internal) | Native vSphere HA |
| Load balancing | Manual | None | Native DRS |

## 1. KVM with CloudStack

### Architecture Diagram

```
                +------------------------+
                |   CloudStack Mgmt      |
                |  (Management Server)   |
                +-----------+------------+
                            |
                            |  SSH / Agent API
                            v
     -------------------------------------------------------
     |                       |                      |
     v                       v                      v
+-----------+         +-----------+          +-----------+
| KVM Host1 |         | KVM Host2 |          | KVM Host3 |
| (libvirt) |         | (libvirt) |          | (libvirt) |
+-----------+         +-----------+          +-----------+
     |                       |                      |
     +--- CloudStack Agent running on each host ----+
     |
     +-- Bridge Interfaces (cloudbr0 / virbr0 / ovsbr0)
     |
     +-- Link Local network for System VM control
     |
     +-- Shared Storage (NFS / SharedMountPoint / OCFS2 / CLVM / GFS2)
```

## 2. XenServer with CloudStack

### Architecture Diagram

```
                +------------------------+
                |    CloudStack Mgmt     |
                |  (Management Server)   |
                +----------+-------------+
                           |
                           | API / SSH
                           v
              +------------+-------------+
              |  XenServer Pool Master   |  ← CloudStack connects ONLY here
              +------------+-------------+
                           |
     ---------------------------------------------------------
     |                       |                       |
     v                       v                       v
+-----------+         +-----------+          +-----------+
|  Host 1   |         |  Host 2   |          |  Host 3   |
|  (in pool)|         |  (in pool)|          |  (in pool)|
+-----------+         +-----------+          +-----------+
     |                       |                       |
     +-- Link Local Network --+-----------------------+
     |      (for System VMs & Virtual Routers)        |
     --------------------------------------------------
                           |
                           v
                +------------------------+
                |   Storage Repositories |
                | (NFS, iSCSI, FC, etc.)|
                +------------------------+
```

## 3. VMware (vSphere/ESXi) with CloudStack

### Architecture Diagram

```
               +-------------------------+
               |   CloudStack Mgmt       |
               +-----------+-------------+
                           |
                           | vCenter API (HTTPS / 443)
                           v
              +------------+------------+
              |      vCenter Server      |
              +------------+-------------+
                           |
        ---------------------------------------------
        |                      |                   |
        v                      v                   v
+-------------+         +-------------+     +-------------+
|   ESXi Host |         |   ESXi Host |     |   ESXi Host |
+-------------+         +-------------+     +-------------+
        |                      |                   |
        +-- Managed via vSwitch/dvSwitch Network --+
        |
        +-- Shared Storage (Datastores: NFS / iSCSI / FC)

System VM Flow:
----------------
CloudStack → vCenter → ESXi host → System VM
(System VMs consume IPs from management CIDR)
SSVM must reach vCenter to copy templates/snapshots.

Notes:
- CloudStack never talks directly to ESXi hosts.
- vCenter is the single control plane.
- DRS/HA functions are handled natively by vSphere.
```

## Comparison Summary

| Aspect | **KVM** | **XenServer** | **VMware** |
|---------|----------|---------------|-------------|
| Connection Model | Agent → Host | API → Pool Master | API → vCenter |
| System VM Network | Link Local | Link Local | Management Network |
| Management Simplicity | Simple (per host) | Centralized (pool) | Enterprise (via vCenter) |
| Licensing | Open-source | Free/Commercial | Proprietary |
| Storage Handling | Mount per host | Managed by pool | Managed by vCenter |
| Best Use Case | Linux-native deployments | Citrix-based infra | Enterprise VMware shops |

## Conclusion

Each hypervisor in CloudStack serves a specific purpose:

- **KVM** → Best for open-source, flexible environments.  
- **XenServer** → Suited for Citrix-based setups needing managed pools.  
- **VMware** → Ideal where vCenter is already in use.
