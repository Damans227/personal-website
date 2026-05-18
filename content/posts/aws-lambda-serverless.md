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

## EC2 vs Lambda

| | EC2 | Lambda |
|---|-----|--------|
| Unit | Virtual server | Function |
| Lifecycle | Continuously running | On-demand, per invocation |
| Scaling | Manual / ASG | Automatic |
| Limits | CPU/RAM you pick | Time — max 15 min execution |
| Billing | Per second running | Per request + compute time |

**The mental shift:** EC2 is a *house* that is always on. Lambda is a *light switch* that turns on only when something triggers it.

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

Lambda is the second event-driven compute option in the model. Putting all three side by side:

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

## Summary

- **Serverless** means you don't manage servers and you pay only for what you use.
- **Lambda** runs functions on demand, scales automatically, and caps each run at 15 minutes.
- **Use Lambda for** event-driven tasks, glue code, APIs, and scheduled jobs.
- **Don't use Lambda for** long-running jobs, custom OS needs, or predictable steady traffic.

---
