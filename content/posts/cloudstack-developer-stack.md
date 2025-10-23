---
title: "CloudStack developer stack"
date: 2025-10-22T15:33:37-05:00
draft: false
tags:
  - cloudstack
  - kvm
---

The CloudStack developer stack connects a macOS host and a Linux Dev VM for local development.
On macOS, developers run the Vue.js UI and connect via SSH tunnel to Jetty on the VM.
The Linux VM hosts the CloudStack Management Server, MySQL, and NFS services.
This setup enables full backend–frontend integration and debugging in isolation.
It simplifies testing and rapid iteration without needing a full production setup.

               
                ┌─────────────────────────────────────────────────────────────┐
                │                  macOS (your laptop)                        │
                │   ┌───────────────────────────────────────────────────────┐ │
                │   │   SSH tunnel → Jetty (localhost:8080/client)          │ │
                │   │   Browser → Vue UI (localhost:8081)                   │ │
                │   └───────────────────────────────────────────────────────┘ │
                └─────────────────────────────────────────────────────────────┘

                                     ⬇ (SSH)

                ┌─────────────────────────────────────────────────────────────┐
                │                  Linux Dev VM (192.168.2.200)               │
                │                                                             │
                │ ┌─────────────────────────────────────────────────────────┐ │
                │ │      CloudStack Management Server (Java monolith)       │ │
                │ │                                                         │ │
                │ │  [Runs in Jetty]                                        │ │
                │ │    - API layer                                          │ │
                │ │    - Orchestration, DB access, plugins, schedulers      │ │
                │ │                                                         │ │
                │ │  [Talks to these via internal APIs / DB / NFS]          │ │
                │ └─────────────────────────────────────────────────────────┘ │
                │                                                             │
                │ ┌────────────────────────────┐  ┌─────────────────────────┐ │
                │ │      MySQL Server          │  │     NFS Server          │ │
                │ │  - CloudStack DB           │  │  - Primary/Secondary    │ │
                │ │  - Used by mgmt server     │  │    storage mounts       │ │
                │ └────────────────────────────┘  └─────────────────────────┘ │
                │                                                             │
                │ ┌─────────────────────────────────────────────────────────┐ │
                │ │        CloudStack Agent (Simulator or KVM)              │ │
                │ │  - Talks to libvirt (in KVM mode)                       │ │
                │ │  - Runs commands from management server                 │ │
                │ │  - Pretends to be a hypervisor if using simulator       │ │
                │ └─────────────────────────────────────────────────────────┘ │
                │                                                             │
                │ ┌─────────────────────────────────────────────────────────┐ │
                │ │                libvirt + KVM (Optional)                 │ │
                │ │  - Only used in real hypervisor mode (KVM/Xen/VMware)   │ │
                │ │  - Not involved in simulator mode                       │ │
                │ └─────────────────────────────────────────────────────────┘ │
                └─────────────────────────────────────────────────────────────┘