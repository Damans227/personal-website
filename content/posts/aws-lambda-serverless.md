---
title: "AWS Lambda: Serverless Compute"
date: 2026-05-17T14:00:00Z
draft: false
tags:
  - AWS
  - Lambda
  - Serverless
---

Serverless is a shift in how you think about compute: instead of provisioning and running servers, you just deploy code. AWS Lambda is the service that pioneered it, and it is the third compute option in the AWS model — alongside EC2 and Fargate.

---

## What serverless means

- A new paradigm — developers don't manage servers, they **just deploy code (functions)**.
- It started as **FaaS** (Function as a Service), pioneered by AWS Lambda.
- The term is now broader: it covers anything *managed* — databases, messaging, storage.
- It **does not mean there are no servers** — just that *you* don't see, provision, or manage them.

Examples of serverless services in AWS: Lambda, Fargate, DynamoDB, S3, Aurora Serverless, SQS, SNS.

---

## Composing a serverless architecture

The core idea goes beyond a single function. You break your app into **small functions**, each triggered by an event, each calling **managed AWS services** for storage, data, and messaging. There are no servers to manage — and done right, your whole app is serverless.

```
Traditional:  1 big app on EC2, you manage everything underneath
Serverless:   many small functions + managed services, glued by events
```

Here is how the pieces compose:

```
User → API Gateway → Lambda functions → S3 / DynamoDB / SQS / SNS / ...
                          │
                          └── each function = one small job
```

You write the *logic*. AWS provides the *plumbing* — triggers, scaling, runtime, and infrastructure.

---

## EC2 vs serverless workloads

| | EC2 (server-based) | Serverless (Lambda + managed) |
|---|---|---|
| **Unit of work** | A running server | A function invocation |
| **Lifecycle** | Always on | Runs on an event, then exits |
| **Scaling** | Manual / ASG | Automatic, instant |
| **Billing** | Per second running — idle still costs | Per invocation — idle is $0 |
| **Execution time** | Unlimited | Max 15 min (Lambda) |
| **State** | Persistent in memory | Stateless — externalize to DB/S3 |
| **OS / runtime control** | Full | None — AWS-managed |
| **Cold start** | None | A slight delay after idle |
| **Best for** | Steady traffic, long processes, full control | Bursty, event-driven, short tasks |
| **Ops burden** | Patching, scaling, monitoring | Almost none |
| **Cost at high steady load** | Cheaper | Can get expensive |
| **Cost at low/spiky load** | Wasteful — idle servers | Very cheap |

**The mental shift:** EC2 is a *house* that is always on. Lambda is a *light switch* that turns on only when something triggers it.

---

## When each wins

**EC2 / containers win for:**

- Steady, predictable traffic.
- Long-running processes (over 15 minutes).
- Real-time work — websockets, low-latency.
- Specialized OS or kernel needs.
- High constant throughput, where serverless cost balloons.

**Serverless wins for:**

- Spiky or unpredictable load.
- Event-driven work — uploads, webhooks, schedules.
- Startups, MVPs, and prototypes.
- Background tasks pulled off the main app.
- When you simply don't want to manage anything.

---

## Benefits of Lambda

- **Pay per use** — request count plus compute time. The free tier covers 1M requests and 400,000 GB-seconds per month.
- **Event-driven** — AWS invokes the function when something happens.
- Integrated with the whole AWS suite.
- Multi-language — Node, Python, Java, C#, Ruby, and custom runtimes.
- Monitoring through CloudWatch.
- Up to **10 GB of RAM** per function — and more RAM also means more CPU and network.

---

## Language support

- **Built-in:** Node.js, Python, Java, C#/.NET, PowerShell, Ruby.
- **Custom runtimes:** Rust, Go, and anything else, via the Lambda Runtime API.
- **Lambda Container Image:** you can deploy a container image *if* it implements the Lambda Runtime API. For arbitrary Docker images, use ECS/Fargate instead.

---

## Mental model

```
   Trigger Event ─────► Lambda Function ─────► Output / side effects
   (S3 upload,          (your code runs        (write to DB, S3,
    HTTP request,        for ≤ 15 min)          call another API)
    cron, etc.)
```

The function sits idle at no cost. An event fires, AWS spins it up, it runs your code, and then it is done. You pay only for those milliseconds.

---

## Example — serverless thumbnail creation

```
   User uploads photo
          │
          ▼
   ┌──────────────┐
   │   S3 Bucket  │  (new image)
   └──────┬───────┘
          │ trigger (event)
          ▼
   ┌──────────────┐
   │    Lambda    │  resizes the image
   └──────┬───────┘
          │
     ┌────┴────┐
     ▼         ▼
   S3        DynamoDB
  (thumb)   (metadata:
            name, size, date)
```

Notice what is **not** in this picture: no EC2, no ELB, no ASG — no servers anywhere. It just runs when needed, scales infinitely, and costs cents.

---

## Where Lambda fits in the architecture

```
            EC2s (steady traffic, long-running app)
                  ↑
       ELB ───────┘

            Lambda (event-driven side jobs)
              ↑
              ├── S3 events (process uploads)
              ├── DynamoDB streams
              ├── API Gateway (HTTP)
              ├── CloudWatch (cron)
              └── SNS/SQS (messaging)
```

Lambda is the event-driven compute option in the model. Putting all three side by side:

| Compute type | When to use it |
|--------------|----------------|
| **EC2** | Always-on, full control, traditional apps |
| **Fargate** (containers) | Containerized apps, no server management |
| **Lambda** (functions) | Short event-driven tasks, ≤ 15 min |

---

## Lambda limits worth remembering

- **Max execution time:** 15 minutes.
- **Max memory:** 10 GB.
- **Deployment package:** 50 MB zipped, 250 MB unzipped — or 10 GB as a container image.
- **Concurrent executions:** 1,000 per account by default, and raisable.
- **/tmp storage:** up to 10 GB.

If your job runs longer than 15 minutes, use Fargate or EC2 — not Lambda.

---

## The mental shift

The real change serverless asks for is in how you frame the problem:

- **EC2 mindset:** "I have a server — what should I run on it?"
- **Serverless mindset:** "I have events — what should happen when they fire?"

---

## Summary

- **Serverless** means composing your app from small functions plus managed services, paying only when something happens, and scaling from zero to near-infinity automatically.
- The trade-off: you give up control and long execution time in exchange for near-zero ops and zero idle cost.
- **Lambda** runs functions on demand, scales automatically, and caps each run at 15 minutes.
- **Use Lambda for** event-driven tasks, glue code, APIs, and scheduled jobs.
- **Don't use Lambda for** long-running jobs, custom OS needs, or predictable steady traffic — reach for EC2 or Fargate there.

---
