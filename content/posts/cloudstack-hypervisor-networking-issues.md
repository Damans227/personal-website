---
title: "CloudStack Hypervisor Network Topology Debugging"
date: 2025-10-22T15:33:37-05:00
draft: false
tags:
  - cloudstack
  - kvm
---

This guide covers how to map and debug CloudStack VM and Virtual Router (VR) network topology using tools like `brctl`, `virsh`, `ip`, `tcpdump`, and `iptables` — all from the hypervisor (host) level.

---

## Mapping Network Topology

### Tools That Work Together

| Tool                     | What it Tells You                                               |
|--------------------------|------------------------------------------------------------------|
| `brctl show`             | Shows which interfaces are connected to which Linux bridges     |
| `virsh domiflist <vm>`   | Maps VMs/VRs to their tap devices and bridges                   |
| `virsh dumpxml <vm>`     | Shows MACs, interface order, bridge assignments (detailed view) |

---

### Example Use

#### Step 1: See which interfaces are on a bridge

```bash
sudo brctl show
```

Example:

```
bridge name     interfaces
--------------  ---------------------
brenp1s0-201     enp1s0.201
                 vnet28
                 vnet41
```

#### Step 2: Map `vnet28` and `vnet41` back to VMs/VRs

```bash
virsh domiflist r-18-VM
virsh domiflist i-2-19-VM
```

Result:

```
r-18-VM:
  vnet28 → brenp1s0-201
  vnet29 → cloud0
  vnet30 → cloudbr0

i-2-19-VM:
  vnet41 → brenp1s0-201
```

### What This Gives You

- Which VMs/VRs are connected to which guest VLANs
- Which bridges connect to physical NICs (e.g., `cloudbr0` → `enp1s0`)
- Traffic flow: **VM ↔ VR ↔ public bridge ↔ NIC ↔ switch**

---

## 🔍 Deeper Inspection with `ip a` and `tcpdump`

### 1. `ip a` – Inspect Interface and IP Configs

#### On Host:

```bash
ip a
```

Check for:
- Bridges: `brenp1s0-201`, `cloud0`, `cloudbr0`
- TAP interfaces: `vnetX`
- Physical NICs: `enp1s0`, `enp1s0.201`

#### Inside VR:

```bash
virsh console r-18-VM
# then:
ip a
ip r
```

---

### 2. `tcpdump` – Live Packet Inspection

#### Monitor ARP, ICMP:
```bash
sudo tcpdump -i brenp1s0-201 arp or icmp
```

#### Monitor VM traffic:
```bash
sudo tcpdump -i brenp1s0-201 host 10.2.1.68
```

#### Monitor VR public traffic:
```bash
sudo tcpdump -i cloudbr0
```

#### Monitor DHCP/DNS:
```bash
sudo tcpdump -i brenp1s0-201 port 67 or port 53
```

---

## Real-World Use Cases

| Problem                | Tool/Command Used                                              |
|------------------------|----------------------------------------------------------------|
| VM can't get IP        | `tcpdump` on `brenp1s0-201` to watch DHCP                      |
| VM can't reach internet| `ip a` inside VR + `tcpdump` on `cloudbr0`                    |
| Ping fails             | `tcpdump` on bridges to trace ICMP                            |
| VR unreachable         | `ip a`, `ip r` inside VR + ARP check from host                 |

---

### Use Cases on Hypervisor

| Goal                            | iptables Usage                                    |
|----------------------------------|---------------------------------------------------|
| View forwarded traffic           | `sudo iptables -L FORWARD -v -n`                  |
| Log traffic from VM             | `sudo iptables -I FORWARD -i vnet41 -j LOG`       |
| Block specific VM traffic       | `sudo iptables -I FORWARD -i vnet41 -j DROP`      |
| Trace VM-to-VR packets          | `sudo iptables -I FORWARD -s 10.2.1.68 -d 10.2.1.1 -j LOG` |
| View NAT rules (usually empty) | `sudo iptables -t nat -L -v -n`                   |