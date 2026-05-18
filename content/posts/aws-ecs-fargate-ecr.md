---
title: "AWS ECS, Fargate & ECR: Running Containers"
date: 2026-05-17T12:00:00Z
draft: false
tags:
  - AWS
  - Containers
  - Compute
---

If your app is packaged as a Docker container, AWS gives you three services that work together to run it: **ECS** to orchestrate containers, **Fargate** to run them without managing servers, and **ECR** to store the images.

---

## ECS — Elastic Container Service

ECS launches Docker containers on AWS. The key thing about it: **you provision and maintain the EC2 instances** — the infrastructure — and AWS handles starting and stopping containers across those EC2s. It integrates with the Application Load Balancer.

```
        ECS Service
            │
       schedules
       containers
            │
   ┌────────┼────────┐
   ▼        ▼        ▼
  EC2      EC2      EC2
 (yours)  (yours)  (yours)
```

The split: **you manage the EC2 instances; AWS manages placing and running containers on them.**

---

## Fargate

Fargate also launches Docker containers on AWS, but with one big difference: **there is no EC2 to provision or manage.** It is the serverless option — you specify the CPU and RAM your container needs, and AWS runs it.

```
       New Container
            │
            ▼
       ┌─────────┐
       │ Fargate │   (no EC2 visible)
       └─────────┘
```

The split: **you manage nothing beyond the container and its CPU/RAM spec; AWS manages everything underneath.**

---

## ECR — Elastic Container Registry

ECR is a private Docker registry on AWS. It stores your Docker images, and ECS or Fargate pulls images from it to run them.

```
   ┌──────┐         ┌──────────┐
   │ ECR  │ ──pull──▶  Fargate  │
   │      │         │  / ECS    │
   │ Img1 │         │           │
   │ Img2 │         └──────────┘
   └──────┘
```

---

## How the three fit together

```
You build Docker image
        │
        ▼
   push to ECR  ←── image storage
        │
        ▼
   ECS or Fargate pulls image
        │
        ▼
   container runs
   (on your EC2s = ECS,
    or AWS-managed = Fargate)
```

You build a Docker image, push it to ECR, and then ECS or Fargate pulls it and runs the container — on your own EC2s with ECS, or on AWS-managed infrastructure with Fargate.

---

## Summary

- **ECS** runs containers on your own EC2s — you own the infrastructure.
- **Fargate** runs containers serverless — no EC2 to manage at all.
- **ECR** is where your Docker images are stored.

---
