---
title: "AWS Batch vs Lambda: Choosing the Right Runner"
date: 2026-05-17T16:00:00Z
draft: false
tags:
  - AWS
  - Batch
  - Serverless
---

Both AWS Batch and Lambda run your code without making you manage servers — but they are built for very different shapes of work. Knowing which is which saves you from forcing a long job into a service that caps out at 15 minutes.

---

## The core difference

| | Lambda | Batch |
|---|--------|-------|
| Built for | Short, event-driven tasks | Long, heavy batch jobs |
| Execution time | ≤ 15 min | Hours or days — no limit |
| Trigger | Events (S3, API, schedule) | Job submission |
| Resources | Up to 10 GB RAM | Any EC2/Fargate size, including GPUs |
| Runs on | Lambda's managed runtime | EC2 or Fargate (you don't manage them, but they exist) |
| Scaling | Instant, per invocation | Queues jobs, then provisions compute |
| Use case | Glue code, reactions, APIs | Data processing, ML training, simulations |

---

## Mental model

```
Lambda:  "Run this small function NOW, fast."
Batch:   "Run these 10,000 heavy jobs whenever there's capacity."
```

---

## What AWS Batch actually is

AWS Batch is a **managed job scheduler** for batch workloads:

- You define **jobs** — a Docker container, a command, and the resources it needs.
- You submit them to a **job queue**.
- Batch provisions EC2 or Fargate, runs the job, and tears the compute down afterward.

```
   You submit jobs ──► Job Queue ──► Compute Environment (EC2/Fargate)
                                          │
                                          ▼
                                    Job runs in container
                                          │
                                          ▼
                                    Output to S3 / DB
```

You don't manage the cluster — Batch sizes it up and down based on the queue.

---

## When to use each

**Lambda:**

- Resize an image when it is uploaded (2 seconds).
- Process a webhook (500 ms).
- An API endpoint behind API Gateway.
- A scheduled cleanup job (5 minutes).
- React to a DynamoDB change.

**Batch:**

- Process 1 TB of genomic data overnight.
- Run a 6-hour ML training job.
- Render 10,000 video frames.
- Build monthly financial reports across years of data.
- Anything that needs GPUs.

---

## Why not just use Lambda for everything?

- Lambda caps at **15 minutes** and **10 GB of RAM**.
- It has no GPUs.
- It is per-invocation, not built around a job queue.
- It gets expensive for sustained heavy compute.

## Why not just use EC2 for batch jobs?

- You would size the cluster manually.
- You would queue jobs manually.
- You would shut things down manually when the work is done.
- Batch automates all of that.

---

## Decision shortcut

| If your job is... | Use |
|-------------------|-----|
| Short, event-triggered, < 15 min | **Lambda** |
| Long, heavy, queue-friendly | **Batch** |
| Long, always-on web app | **EC2 / Fargate** |
| Short, containerized, on-demand | **Fargate task** |

---

## Summary

- **Lambda** is serverless **functions** for short, event-driven tasks.
- **Batch** is a managed **job runner** for long, heavy, queued workloads.
- Both are "serverless-ish" — you don't manage servers — but Lambda is for *reactions*, while Batch is for *workloads*.

---
