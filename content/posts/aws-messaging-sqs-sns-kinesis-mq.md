---
title: "AWS Messaging: SQS, SNS, Kinesis, and Amazon MQ"
date: 2026-05-17T22:00:00Z
draft: false
tags:
  - AWS
  - Messaging
  - Integration
---

Once an app grows past a single service, the pieces need to talk to each other — but you don't want them tightly coupled. AWS's messaging services are the glue: they let one part of your app hand off work or events to another without either side having to know much about the other.

---

## The mental map

```
┌─────────────────────────────────────────────────────────┐
│  PATTERN 1: One-to-one (queue)                          │
│  • SQS         (producer → queue → one consumer)        │
├─────────────────────────────────────────────────────────┤
│  PATTERN 2: One-to-many (pub/sub)                       │
│  • SNS         (publisher → topic → many subscribers)   │
├─────────────────────────────────────────────────────────┤
│  PATTERN 3: Real-time data streams                      │
│  • Kinesis     (continuous data → process → store)      │
├─────────────────────────────────────────────────────────┤
│  PATTERN 4: Industry-standard protocols                 │
│  • Amazon MQ   (managed RabbitMQ/ActiveMQ for migration)│
└─────────────────────────────────────────────────────────┘
```

---

## SQS — Simple Queue Service

SQS is a managed message **queue**. Producers send messages in; consumers poll them out. Each message is read by **one** consumer and then deleted.

```
Producers ──send──► [SQS Queue] ──poll──► Consumers
```

**Key facts:**

- AWS's oldest service — it has been around 10+ years.
- **Serverless** and fully managed.
- Scales from 1 to 10,000+ messages per second.
- Message retention: 4 days by default, up to 14 days.
- No limit on queue size.
- A message is **deleted once a consumer reads it** — one delivery.
- Under 10 ms latency.
- Used to **decouple** application tiers.

**Why decouple?** If web servers and video processors are wired directly together, a slow video processor blocks user uploads. With SQS between them, web servers PUT a job and move on, and processors pull work at their own pace. The two sides scale independently.

```
WEB SERVERS ──PUT──► [SQS] ──pull──► VIDEO PROCESSORS
   (ASG)                                 (ASG, scales
                                          based on queue depth)
```

Reach for SQS to decouple tiers, smooth out spiky load, or run background job processing.

---

## SNS — Simple Notification Service

SNS is **pub/sub** — one message goes to *many* subscribers at once.

```
Publisher ──► [SNS Topic] ──► Subscriber 1 (SQS)
                         ──► Subscriber 2 (Lambda)
                         ──► Subscriber 3 (Email)
                         ──► Subscriber 4 (HTTP endpoint)
```

**Why it exists:** without SNS, a Buying Service would have to send the same message to Email, Fraud, and Shipping individually — direct integration with each. With SNS, it publishes *once* to a topic, and every subscriber gets it.

**Key facts:**

- Up to 12.5M subscriptions per topic, with a max of 100,000 topics.
- One publisher → one topic; one topic → many subscribers.
- **Every subscriber gets every message.**
- Subscribers can be SQS, Lambda, Firehose, email, SMS, mobile push, or HTTP(S).

Use SNS for event fan-out, notifications, and broadcasting to many systems.

---

## SQS vs SNS — the key distinction

| | SQS | SNS |
|---|-----|-----|
| Pattern | Queue (one-to-one) | Pub/sub (one-to-many) |
| Message flow | Pulled by consumer | Pushed to subscribers |
| Delivery | One consumer reads, then deleted | All subscribers get a copy |
| Retention | Up to 14 days | No retention — it's a push |
| Mental model | "Work queue" | "Broadcast" |

**The common combo: SNS + SQS fan-out.** An SNS topic with SQS queues as subscribers. Each consumer gets its own queue, processes independently, and benefits from SQS's durability. Best of both.

---

## Kinesis — real-time data streams

Kinesis is AWS's managed service for **real-time big data streaming** — collecting, processing, and analyzing data as it arrives.

```
Sources                Stream                  Destinations
───────                ──────                  ────────────
Click streams ──┐                            ┌── S3
IoT devices   ──┼──► Kinesis Data Streams ──► Firehose ──┼── Redshift
Metrics/logs  ──┘                            └── ...
```

- **Kinesis Data Streams** ingests the stream.
- **Kinesis Data Firehose** loads streams into destinations such as S3 or Redshift.

Use it for application logs, IoT telemetry, clickstream analytics, real-time metrics, or ML feature pipelines. The one-line takeaway: **Kinesis = real-time big data streaming.**

### SQS vs Kinesis (often confused)

| | SQS | Kinesis |
|---|-----|---------|
| Job | Decouple apps (queue) | Stream big data |
| Model | One consumer per message | Many consumers can replay |
| Retention | 4–14 days | Up to 365 days |
| Order | Best-effort (FIFO opt-in) | Per-shard ordering |
| Scale | Auto | Manual sharding |

---

## Amazon MQ — managed message broker

Amazon MQ is managed **RabbitMQ** and **ActiveMQ** — for apps that use industry-standard protocols like MQTT, AMQP, STOMP, Openwire, and WSS.

**Why it exists:**

- SQS and SNS use **AWS-proprietary protocols**.
- Legacy on-prem apps speak open protocols.
- If you are migrating to AWS, you don't want to rewrite — Amazon MQ lets you keep the protocol the same.

**Key facts:**

- Runs on actual servers — not serverless, unlike SQS/SNS.
- Doesn't scale as far as SQS/SNS.
- Can run **Multi-AZ with failover**.
- Provides **both queue (SQS-like) and topic (SNS-like) features**.

Use it when you are lifting an on-prem app that already speaks MQTT/AMQP/STOMP into AWS, without rewriting it.

---

## How they fit in our architecture

```
                Users
                  │
                  ▼
                ELB
                  │
                  ▼
              EC2s / Lambda
              (web tier)
                  │
       ┌──────────┼──────────┬──────────┐
       │          │          │          │
       ▼          ▼          ▼          ▼
     RDS        S3        SQS        SNS Topic
   (data)     (blobs)   (jobs)     (events)
                          │            │
                          ▼            ├──► Email
                     Worker EC2s       ├──► Lambda
                     (process at       └──► SQS (fan-out)
                      own pace)
```

---

## Decision shortcuts

| Need | Service |
|------|---------|
| One worker handles each message | **SQS** |
| Broadcast a message to many systems | **SNS** |
| Fan-out — each consumer gets the full message stream, durably | **SNS + SQS** |
| Real-time streaming data (logs, IoT, clicks) | **Kinesis** |
| Migrate an on-prem app using RabbitMQ / ActiveMQ | **Amazon MQ** |

---

## Summary

- **SQS** is a queue with one consumer per message — use it to decouple tiers.
- **SNS** is pub/sub — broadcast one message to many subscribers.
- **Kinesis** is for real-time data streaming at scale.
- **Amazon MQ** is managed RabbitMQ/ActiveMQ for lift-and-shift migrations.
- These are the plumbing services that let parts of your app communicate without being directly wired together. Pick by communication *pattern*, not by service name.

---
