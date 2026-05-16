---
title: "AWS ELB & ASG: Load Balancing and Auto Scaling"
date: 2026-05-16T18:00:00Z
draft: false
tags:
  - AWS
  - Networking
  - Compute
---

A single EC2 instance is a single point of failure. To make an app **highly available** and **elastic** — able to handle variable load and survive failures automatically — AWS gives you two services that work hand in hand: **ELB** and **ASG**.

---

## The two pieces

| Service | What it does | Question it answers |
|---------|--------------|---------------------|
| **ELB** | Distributes incoming traffic across multiple EC2s | "How do users reach my fleet?" |
| **ASG** | Adds, removes, and replaces EC2s based on demand or health | "How big should my fleet be?" |

They are almost always used **together**.

---

## Mental model

```
Users ──> ELB ──> [EC2, EC2, EC2, ...]  ← fleet managed by ASG
              spreads traffic         adds/removes/replaces instances
```

- **ELB is the traffic cop** — it routes requests.
- **ASG is the fleet manager** — it right-sizes the fleet and heals it.

---

## ELB — Elastic Load Balancer

**What it does:**

- Presents a single, stable DNS name in front of a constantly changing fleet.
- Runs health checks, and only routes to healthy instances.
- Spreads traffic across Availability Zones.
- Terminates SSL/TLS, offloading that work from the EC2s.

**Four types:**

| Type | Layer | Use |
|------|-------|-----|
| **ALB** (Application LB) | Layer 7 (HTTP/HTTPS) | Web apps, path/host-based routing, microservices |
| **NLB** (Network LB) | Layer 4 (TCP/UDP) | Extreme performance, static IPs, non-HTTP traffic |
| **GLB** (Gateway LB) | Layer 3 | Routing through third-party appliances such as firewalls |
| **CLB** (Classic LB) | L4 / L7 | Legacy — don't use it for anything new |

For web apps, the **default choice is ALB**.

**Key ALB features:**

- **Path-based routing** → `/api/*` goes to the API fleet, `/img/*` goes to the image fleet.
- **Host-based routing** → `api.app.com` and `www.app.com` route to different fleets.
- **Target groups** → groupings of instances, containers, or Lambdas that the load balancer sends traffic to.

---

## ASG — Auto Scaling Group

**What it does:**

- Maintains a defined number of EC2s — **min, desired, and max**.
- Launches replacements when instances fail their health checks.
- Scales out and in based on metrics such as CPU, request count, or custom signals.
- Spreads instances across Availability Zones automatically.

**Key settings:**

- **Min / Desired / Max** — the bounds on fleet size.
- **Launch Template** — the AMI, instance type, security group, IAM role, and user data to use for new instances.
- **Health checks** — at the EC2 level (is the VM alive?) and/or the ELB level (is the app responding?).
- **Scaling policies:**
  - **Target tracking** → "keep CPU at 50%." The most common and simplest.
  - **Step scaling** → "if CPU > 70%, add 2 instances."
  - **Scheduled** → "scale up every weekday at 9am."

---

## How they work together — a running example

Consider a **photo app** with a typical weekday traffic pattern:

1. The ASG is configured with `min=2`, `max=10`, `desired=2`.
2. An ALB sits in front, registered to the ASG's target group.
3. **Morning:** traffic spikes and CPU hits 70% on both EC2s.
4. The ASG launches EC2 #3 from the launch template.
5. The new EC2 boots, passes its health check, and the ALB starts sending it traffic.
6. **Evening:** traffic drops, CPU falls to 20%, and the ASG terminates the extra EC2s.
7. **Middle of the night:** EC2 #2 crashes, fails its health check, and the ASG terminates it and launches a replacement.

The app stayed up the whole time, with no human involved.

---

## Sticky sessions (an ALB feature)

- Normally each request can go to any EC2 — the app is treated as stateless.
- **Sticky sessions** make the same user always land on the same EC2, tracked via a cookie.
- This is useful when session state lives on the instance, but **stateless apps scale better** — keep session state in Redis or a database instead.

---

## Cross-zone load balancing

- **Enabled** → the load balancer spreads traffic evenly across all instances, regardless of which AZ they are in.
- **Disabled** → traffic is spread evenly across AZs first, then across the instances within each AZ.
- On ALB it is on by default and free. On NLB it is off by default and costs extra.

---

## Key principles

- An **ELB needs at least 2 AZs** to be useful — a single AZ means no high availability.
- An **ASG spanning multiple AZs** survives the loss of an entire AZ.
- **Health checks are critical** — bad health checks send traffic to dead instances.
- Use **launch templates**, not the older launch configurations.
- **Stateless EC2s scale best** — push state out to S3, RDS, or Redis.
- An **ASG replaces, it does not repair** — treat instances as cattle, not pets.

---

## Summary

- **ELB** spreads traffic across a fleet; **ASG** manages the size and health of that fleet.
- Together they give you an app that is both elastic and self-healing.
- Use an ALB for web apps, span multiple AZs for real availability, and keep your instances stateless so the fleet can grow, shrink, and heal freely.

---
