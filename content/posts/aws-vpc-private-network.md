---
title: "AWS VPC: Your Private Network in the Cloud"
date: 2026-05-18T12:00:00Z
draft: false
tags:
  - AWS
  - Networking
  - VPC
---

Every resource you run in AWS — an EC2 instance, an RDS database, a load balancer — sits inside a network. That network is the VPC: a private, isolated slice of the AWS cloud that you control. Understanding it means understanding how traffic actually reaches your resources, and how you keep them protected. The single most important distinction to walk away with is security groups versus NACLs, so that gets special attention below.

---

## The mental map

```
┌─────────────────────────────────────────────────────────┐
│  THE NETWORK ITSELF                                     │
│  • VPC          (your private network, regional)        │
│  • Subnets      (partitions of the VPC, per-AZ)         │
│  • Route Tables (define what can reach what)            │
├─────────────────────────────────────────────────────────┤
│  INTERNET ACCESS                                        │
│  • Internet Gateway (IGW)  (public subnet → internet)   │
│  • NAT Gateway/Instance    (private subnet → internet)  │
├─────────────────────────────────────────────────────────┤
│  FIREWALLS                                              │
│  • Security Groups  (instance-level, stateful, allow)   │
│  • NACL             (subnet-level, stateless, allow+deny)│
├─────────────────────────────────────────────────────────┤
│  CONNECT VPCs / AWS SERVICES                            │
│  • VPC Peering      (connect 2 VPCs)                    │
│  • VPC Endpoints    (private access to AWS services)    │
│  • PrivateLink      (expose service to 1000s of VPCs)   │
│  • Transit Gateway  (hub connecting many VPCs)          │
├─────────────────────────────────────────────────────────┤
│  CONNECT ON-PREM                                        │
│  • Site-to-Site VPN  (encrypted, over internet)         │
│  • Direct Connect    (physical private line)            │
│  • Client VPN        (your laptop → VPC)                │
├─────────────────────────────────────────────────────────┤
│  MONITORING                                             │
│  • VPC Flow Logs    (network traffic logs)              │
└─────────────────────────────────────────────────────────┘
```

---

## IP addresses first

A bit of foundation before the network itself:

- **IPv4 public** — reachable on the internet. An EC2 instance gets a *new* public IP on stop/start.
- **IPv4 private** — internal network only (e.g. `192.168.1.1`). Stays *fixed* across stop/start.
- **Elastic IP (EIP)** — a *fixed* public IPv4 you attach to an instance. It costs money when not in use.
- **IPv6** — all public in AWS, free, and effectively unlimited.

Note: all public IPv4 now costs roughly $0.005/hr, including EIPs.

---

## VPC & subnets — the core

- **VPC (Virtual Private Cloud):** your own private network in AWS. It is **regional** — it spans a whole region.
- **Subnet:** a partition of the VPC, tied to **one AZ**.
  - A **public subnet** is reachable from the internet.
  - A **private subnet** is not.
- **Route tables** decide what each subnet can reach — the internet, other subnets, and so on.

```
Region
└── VPC (CIDR 10.0.0.0/16)
    ├── AZ-1
    │   ├── Public subnet
    │   └── Private subnet
    └── AZ-2
        ├── Public subnet
        └── Private subnet
```

In a typical web app, the load balancer goes in a public subnet, while the application servers and database sit in private subnets — exposed only as much as they need to be.

---

## Internet access

**Internet Gateway (IGW):**

- Attached at the **VPC level**.
- Public subnets have a route to it, so their instances can reach the internet.

**NAT Gateway (AWS-managed) / NAT Instance (self-managed):**

- Lets **private subnet** instances reach the internet *outbound* — for updates, API calls — while staying unreachable *inbound*.
- "Private, but can still phone out."

```
www
 │
IGW ──── Public Subnet (EC2 can reach internet directly)
 │
 └─ NAT ── Private Subnet (EC2 reaches internet via NAT, but internet can't reach it)
```

---

## Firewalls — Security Groups vs NACLs

This is the distinction worth knowing cold.

| | Security Group | NACL |
|---|----------------|------|
| Level | **Instance** (EC2/ENI) | **Subnet** |
| Rules | **Allow only** | **Allow AND deny** |
| State | **Stateful** — return traffic auto-allowed | **Stateless** — must allow return traffic explicitly |
| Evaluation | All rules evaluated together | Rules processed in number order |
| Targets | IPs + other security groups | IPs only |

**Mental model:**

- A **security group** is the bouncer at each instance's door — stateful, an allow-list.
- A **NACL** is the guard at the subnet's gate — stateless, and able to explicitly ban.

**Stateful vs stateless** is the heart of it:

- With a security group, you allow inbound port 443 and the response goes out automatically.
- With a NACL, you allow inbound 443 and you *also* must allow the outbound response port.

---

## Connecting VPCs & AWS services

**VPC Peering:**

- Connects two VPCs privately, so they behave as one network.
- The CIDR ranges **must not overlap**.
- **Not transitive** — A↔B and B↔C does *not* give A↔C. You must peer A↔C directly.

**VPC Endpoints:**

- Access AWS services (S3, DynamoDB, etc.) over AWS's **private network** instead of the public internet.
- More secure, with lower latency.
- Two types: **Gateway** (S3 and DynamoDB only) and **Interface/ENI** (most services).

**PrivateLink (VPC Endpoint Services):**

- The most secure and scalable way to expose *your* service to thousands of other VPCs.
- No peering, IGW, or NAT needed.
- Uses a Network Load Balancer on the service side and an ENI on the consumer side.

**Transit Gateway:**

- A **hub** that connects thousands of VPCs plus on-prem in a star topology.
- Solves the "peering mesh gets insane" problem — peering is not transitive, so N VPCs would need N² connections.
- One gateway, transitive routing. Works with Direct Connect and VPN.

---

## Connecting on-premises to AWS

**Site-to-Site VPN:**

- On-prem data center → AWS, **encrypted**, over the **public internet**.
- On-prem side: a **Customer Gateway (CGW)**. AWS side: a **Virtual Private Gateway (VGW)**.
- Quick to set up.

**Direct Connect (DX):**

- A **physical private line** between your data center and AWS.
- Private, fast, and secure — it **doesn't touch the public internet**.
- Takes **at least a month** to establish, because of the physical cabling.

**Client VPN:**

- Connects *your laptop* (via OpenVPN) into your VPC.
- Lets you reach EC2 instances by private IP, as if you were inside the network.
- Goes over the public internet.

```
Site-to-Site VPN  = office network ↔ AWS (encrypted, public internet)
Direct Connect    = office network ↔ AWS (private physical line, slow to set up)
Client VPN        = your laptop ↔ AWS (OpenVPN)
```

---

## VPC Flow Logs

- Capture **IP traffic info** for VPCs, subnets, and ENIs.
- For monitoring and troubleshooting connectivity — subnet↔internet, subnet↔subnet, and so on.
- Also capture AWS-managed interfaces such as ELB, RDS, ElastiCache, and Aurora.
- Output to **S3, CloudWatch Logs, or Data Firehose**.

---

## A typical VPC layout

Putting the pieces together, a standard three-tier web app maps onto a VPC like this:

```
Region
└── VPC
    ├── Public subnets  →  ELB (internet-facing)
    │                      via Internet Gateway
    │
    └── Private subnets →  EC2s (the app)  ← Security Groups
                           RDS (database)
                           │
                           ├── NAT Gateway (outbound updates)
                           └── VPC Endpoint → S3 (private, no internet)
```

The load balancer sits in public subnets (internet-facing). The application servers and database sit in private subnets, protected from inbound traffic. NAT lets them phone out for updates, and a VPC endpoint lets them reach S3 without ever touching the public internet.

---

## Decision shortcuts

| Need | Service |
|------|---------|
| A private network for my resources | **VPC** |
| Partition by AZ | **Subnets** |
| Public subnet → internet | **Internet Gateway** |
| Private subnet → internet (outbound only) | **NAT Gateway** |
| Firewall at instance level | **Security Group** |
| Firewall at subnet level (with deny) | **NACL** |
| Connect two VPCs | **VPC Peering** |
| Reach S3/DynamoDB privately | **VPC Endpoint** |
| Connect thousands of VPCs | **Transit Gateway** |
| Office ↔ AWS, encrypted over internet | **Site-to-Site VPN** |
| Office ↔ AWS, private physical line | **Direct Connect** |
| Laptop ↔ AWS | **Client VPN** |
| Log network traffic | **VPC Flow Logs** |

---

## Quick reference

- **VPC** is a regional private network; a **subnet** is an AZ-level partition.
- **IGW** is public internet access (at the VPC level); **NAT** is private subnets' outbound internet.
- **Security Group** is instance-level, **stateful**, allow-only.
- **NACL** is subnet-level, **stateless**, allow + deny.
- **VPC Peering** is non-transitive, with no overlapping CIDR.
- **VPC Endpoint** is private access to AWS services.
- **Site-to-Site VPN** is encrypted over the internet; **Direct Connect** is a private physical line.
- **Transit Gateway** is a hub for thousands of VPCs, with transitive routing.

---

## Summary

- A **VPC** is your private network; **subnets** carve it up by AZ.
- **Gateways** — IGW and NAT — control internet access.
- **Security groups and NACLs** are the two firewall layers, the first stateful and instance-level, the second stateless and subnet-level.
- A family of services — **Peering, Endpoints, PrivateLink, Transit Gateway, VPN, and Direct Connect** — connect your VPC to other VPCs, to AWS services, and to on-prem.

---
