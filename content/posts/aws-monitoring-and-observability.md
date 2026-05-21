---
title: "AWS Monitoring & Observability: CloudWatch, CloudTrail, X-Ray and More"
date: 2026-05-18T10:00:00Z
draft: false
tags:
  - AWS
  - Monitoring
  - Observability
---

Once an app is running on AWS, the next question is: how do you know if it is actually okay? AWS gives you a handful of services to watch performance, audit who did what, trace requests across systems, and check the health of AWS itself.

---

## The mental map

```
┌─────────────────────────────────────────────────────────┐
│  WATCH performance & react                              │
│  • CloudWatch Metrics  (numbers: CPU, network, billing) │
│  • CloudWatch Alarms   (trigger on a metric)            │
│  • CloudWatch Logs     (collect log files)              │
│  • EventBridge         (react to events / schedule)     │
├─────────────────────────────────────────────────────────┤
│  AUDIT who did what                                     │
│  • CloudTrail          (API call history / audit)       │
├─────────────────────────────────────────────────────────┤
│  TRACE & analyze app behavior                           │
│  • X-Ray               (trace requests across services) │
│  • CodeGuru            (ML code review + profiling)     │
├─────────────────────────────────────────────────────────┤
│  CHECK service health                                   │
│  • Health Dashboard (Service)  (all AWS, all regions)   │
│  • Health Dashboard (Account)  (events impacting YOU)   │
└─────────────────────────────────────────────────────────┘
```

---

## CloudWatch — the core monitoring service

### Metrics

CloudWatch Metrics are **numbers tracked over time** — CPUUtilization, NetworkIn, and so on. Every AWS service emits metrics, all timestamped, and you can build dashboards from them.

**Important metrics to know:**

- **EC2:** CPU, status checks, and network — *not RAM*, since RAM isn't a default metric. The default frequency is every 5 minutes; **Detailed Monitoring** (paid) drops it to every 1 minute.
- **EBS:** disk reads and writes.
- **S3:** `BucketSizeBytes`, `NumberOfObjects`, `AllRequests`.
- **Billing:** Total Estimated Charge — only in `us-east-1`.
- **Custom metrics:** push your own.

### Alarms

CloudWatch Alarms **trigger actions when a metric crosses a threshold.**

- Actions: **Auto Scaling** (change the desired count), **EC2 Actions** (stop, terminate, reboot, recover), or **SNS notification**.
- States: **OK**, **INSUFFICIENT_DATA**, **ALARM**.
- Classic example: a **billing alarm** that notifies you when spend exceeds $X.

### Logs

CloudWatch Logs collects log files from Beanstalk, ECS, Lambda, CloudTrail, EC2 and on-prem (via the CloudWatch agent), and Route 53.

- **EC2 needs the CloudWatch agent installed** to push logs — by default, no EC2 logs flow to CloudWatch. It also needs the right IAM permissions.
- The agent works on-prem too.
- This enables real-time log monitoring.

---

## EventBridge (formerly CloudWatch Events)

EventBridge lets you **react to events or run things on a schedule.**

```
Source (S3 upload, EC2 state, schedule, CloudTrail API call)
   │
   ▼
EventBridge (rule matches)
   │
   ▼
Target (Lambda, SQS, SNS, Step Functions, ECS, etc.)
```

**Two trigger types:**

- **Schedule (cron)** → "every hour, run this Lambda."
- **Event pattern** → "when the root user signs in, send an SNS alert."

**Event buses:**

- **Default** — AWS service events.
- **Partner** — SaaS apps like Zendesk and Datadog.
- **Custom** — your own apps.

It also has a schema registry and supports archiving and replaying events.

**Mental model:** EventBridge is the "if this AWS thing happens, then do that" router — the backbone of event-driven automation.

---

## CloudTrail — audit log

CloudTrail records **every API call or action in your AWS account** — who did what, when, and from where.

```
Console / SDK / CLI / IAM users & roles
   │
   ▼
CloudTrail (records the action)
   │
   ├──► CloudWatch Logs
   └──► S3 bucket
```

**Key facts:**

- **Enabled by default.**
- Used for governance, compliance, and audit.
- Applies to **all regions** by default, or one region if you choose.
- The rule of thumb: **if a resource is deleted in AWS, check CloudTrail first** — it tells you who deleted it and when.

**CloudWatch vs CloudTrail (commonly confused):**

- **CloudWatch** = performance, metrics, logs — *how is it doing?*
- **CloudTrail** = audit, API history — *who did what?*

---

## X-Ray — distributed tracing

X-Ray traces a request as it flows through your services.

The problem it solves: in a microservice or distributed app, a single request hits EC2 → DynamoDB → SNS → and so on. When it is slow, *where* is the bottleneck? Logs alone can't tell you. X-Ray draws a visual service map with latency at each hop.

```
Client → EC2 (70ms) → DynamoDB (30ms)
                    → SNS (43ms)
```

**Advantages:** troubleshoot bottlenecks, understand dependencies, pinpoint service issues, find errors and exceptions, check SLAs, see where you are being throttled, and identify impacted users.

Use it whenever you have a distributed or microservice app and can't tell which service is slow.

---

## CodeGuru — ML-powered code analysis

CodeGuru has two parts, covering two phases of the dev lifecycle:

```
Coding ──► Build & Test ──► Deploy ──► Measure
   │                                       │
CodeGuru Reviewer                    CodeGuru Profiler
(static analysis,                    (runtime performance,
 pre-prod)                            production)
```

**CodeGuru Reviewer** — automated code reviews via static analysis:

- Finds bugs, security vulnerabilities, and resource leaks.
- ML model trained on millions of code reviews.
- Supports Java and Python.
- Integrates with GitHub, Bitbucket, and CodeCommit.

**CodeGuru Profiler** — runtime performance in production:

- Finds code inefficiencies and excessive CPU use.
- Reduces CPU and compute costs.
- Provides heap summaries and anomaly detection.
- Works on AWS or on-prem.

**Mental model:** Reviewer checks your code *before* prod; Profiler watches your code *in* prod.

---

## Health Dashboards — is AWS itself okay?

There are two health dashboards, and they are often confused:

**Service Health Dashboard** (formerly the AWS Service Health Dashboard):

- The status of **all AWS services across all regions** — the *general* AWS status.
- Historical info per day, plus an RSS feed.
- Answers: "is AWS having an outage somewhere?"

**Account Health Dashboard** (formerly the Personal Health Dashboard, PHD):

- **Events that specifically impact YOUR resources.**
- Personalized alerts and remediation guidance.
- Can aggregate across an entire AWS Organization.
- Answers: "is anything AWS is doing going to affect *me*?"

The distinction: **Service = global AWS status. Account = personalized to your account.**

---

## How they fit together

```
                  Your Application
                        │
   ┌────────────────────┼────────────────────┐
   │                    │                     │
   ▼                    ▼                     ▼
CloudWatch         CloudTrail              X-Ray
(metrics, logs,    (who did what,          (trace requests
 alarms)            audit trail)            across services)
   │
   ▼
EventBridge ──► automate reactions (Lambda, SNS, etc.)

Meanwhile:

Health Dashboards ──► is AWS itself healthy? (service-wide / your account)
CodeGuru ──► is your code good & efficient?
```

---

## Decision shortcuts

| Need | Service |
|------|---------|
| Monitor CPU/network/performance | **CloudWatch Metrics** |
| Get notified when a metric crosses a threshold | **CloudWatch Alarms** |
| Collect and search application logs | **CloudWatch Logs** |
| React to an event or run on a schedule | **EventBridge** |
| Find out who deleted/changed a resource | **CloudTrail** |
| Debug slowness across microservices | **X-Ray** |
| Automated code review / find bugs | **CodeGuru Reviewer** |
| Find runtime performance issues in prod | **CodeGuru Profiler** |
| Is AWS having an outage? | **Service Health Dashboard** |
| Is AWS doing something that affects *my* resources? | **Account Health Dashboard** |

---

## The big mental split

```
PERFORMANCE  →  CloudWatch (metrics, logs, alarms)
AUTOMATION   →  EventBridge (react to events)
AUDIT        →  CloudTrail (who did what)
TRACING      →  X-Ray (where's the bottleneck)
CODE QUALITY →  CodeGuru (review + profile)
AWS STATUS   →  Health Dashboards (service-wide / your account)
```

---

## Summary

- **CloudWatch** tells you how your app is performing — metrics, alarms, and logs.
- **EventBridge** reacts automatically — to events or on a schedule.
- **CloudTrail** tells you who did what — the audit trail.
- **X-Ray** tells you where requests slow down in a distributed system.
- **CodeGuru** reviews your code (Reviewer) and profiles it in production (Profiler).
- **Health Dashboards** tell you whether AWS itself is healthy — globally, or for your account specifically.
- Pick by the question you are asking: performance, audit, tracing, automation, code quality, or AWS status.

---
