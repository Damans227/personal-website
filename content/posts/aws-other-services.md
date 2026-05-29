---
title: "AWS Other Services: A Grab Bag Worth Knowing About"
date: 2026-05-18T18:00:00Z
draft: false
tags:
  - AWS
  - Services
---

AWS has a long tail of services that don't slot neatly into compute, storage, networking, or databases — but you'll bump into them sooner or later. This post is a quick tour of that grab bag: virtual desktops, mobile dev kits, backup and disaster recovery, migration tooling, workflow orchestration, and a few specialised oddities like satellite ground stations.

---

## The mental map

```
┌─────────────────────────────────────────────────────────┐
│  END-USER COMPUTING                                     │
│  • WorkSpaces     (managed Windows/Linux desktops)      │
│  • AppStream 2.0  (stream apps to a browser)            │
├─────────────────────────────────────────────────────────┤
│  APP DEVELOPMENT                                        │
│  • IoT Core               (connect IoT devices)         │
│  • AppSync                (GraphQL backend)             │
│  • Amplify                (full-stack mobile/web)       │
│  • Infrastructure Composer(visual serverless design)    │
│  • Device Farm            (mobile/web test farm)        │
├─────────────────────────────────────────────────────────┤
│  BACKUP & DISASTER RECOVERY                             │
│  • AWS Backup                  (centralised backups)    │
│  • Elastic Disaster Recovery   (server replication)     │
├─────────────────────────────────────────────────────────┤
│  MIGRATION                                              │
│  • DataSync                       (move data to AWS)    │
│  • Application Discovery Service  (inventory on-prem)   │
│  • Application Migration Service  (lift-and-shift)      │
│  • Migration Evaluator            (business case)       │
│  • Migration Hub                  (central tracking)    │
├─────────────────────────────────────────────────────────┤
│  RESILIENCE & WORKFLOWS                                 │
│  • Fault Injection Simulator (chaos engineering)        │
│  • Step Functions            (serverless workflows)     │
├─────────────────────────────────────────────────────────┤
│  SPECIALISED                                            │
│  • Ground Station  (satellite operations)               │
│  • Pinpoint        (marketing comms)                    │
└─────────────────────────────────────────────────────────┘
```

---

## End-user computing

### Amazon WorkSpaces

WorkSpaces is **Desktop as a Service** — a managed way to provision Windows or Linux desktops in the cloud. It eliminates the headache of running on-premise VDI (Virtual Desktop Infrastructure), scales to thousands of users quickly, integrates with KMS for secure data, and bills pay-as-you-go on monthly or hourly rates.

It works well across regions — a company can give California staff WorkSpaces in `us-east-1` and Paris staff WorkSpaces in `eu-west-2`, all from the same managed service.

### Amazon AppStream 2.0

AppStream 2.0 streams **desktop applications** to any device that has a web browser — no VDI to connect to, no infrastructure to provision.

**AppStream vs WorkSpaces:**

- **WorkSpaces** is a fully managed VDI: users connect to a virtual desktop and open applications there. Desktops can be on-demand or always-on.
- **AppStream 2.0** streams a single application to a browser. Works on any device. You can configure an instance type per application (CPU, RAM, GPU).

---

## App development

### AWS IoT Core

IoT — "Internet of Things" — is the network of internet-connected devices that collect and transfer data. **AWS IoT Core** is the on-ramp: serverless, secure, scaling to billions of devices and trillions of messages. Applications can communicate with devices even when those devices are temporarily disconnected, and IoT Core integrates with the rest of AWS (Lambda, S3, SageMaker, and so on) to gather, process, analyse, and act on the data.

### AWS AppSync

AppSync stores and syncs data across mobile and web apps in real time, using **GraphQL**. Client code can be generated automatically. It integrates with DynamoDB and Lambda, supports real-time subscriptions and offline sync (the replacement for Cognito Sync), and offers fine-grained security. **AWS Amplify** uses AppSync under the hood.

### AWS Amplify

Amplify is a set of tools and services for building and deploying **scalable full-stack web and mobile applications** — authentication, storage, APIs (REST and GraphQL), CI/CD, pub/sub, analytics, AI/ML predictions, monitoring, and source code from AWS, GitHub, and others. The backend is composed from Amazon S3, Cognito, AppSync, API Gateway, Lambda, DynamoDB, SageMaker, and Lex, with Amplify Studio as the visual frontend.

### AWS Infrastructure Composer

Infrastructure Composer lets you **visually design and build serverless applications** on AWS without needing deep AWS expertise. You configure how resources interact, and it generates Infrastructure-as-Code (CloudFormation). It can also import existing CloudFormation or SAM templates so you can visualise them.

### AWS Device Farm

Device Farm tests web and mobile apps against **real mobile devices and tablets**, plus desktop browsers — not emulators. You can run tests concurrently on multiple devices to speed things up, and configure device settings like GPS, language, Wi-Fi, and Bluetooth. The output is reports, logs, and screenshots.

---

## Backup and disaster recovery

### AWS Backup

AWS Backup is a fully managed service to **centrally manage and automate backups** across AWS services. It supports on-demand and scheduled backups, point-in-time recovery, retention periods and lifecycle management, backup policies, and both **cross-region** and **cross-account** backup (via AWS Organizations). It covers EC2, EBS, DynamoDB, RDS, EFS, Aurora, FSx, and Storage Gateway, with backups stored in S3.

### Disaster recovery strategies

There is a spectrum of DR approaches, from cheap and slow to expensive and instant:

- **Backup and Restore** — keep backups in S3. Cheapest. Slowest recovery — you restore from scratch.
- **Pilot Light** — core functions of the app run continuously in AWS, ready to scale up. Minimal setup, faster recovery.
- **Warm Standby** — a full version of the app runs in AWS at minimum size, ready to scale up.
- **Multi-Site / Hot-Site** — a full version of the app runs in AWS at full size. Most expensive, instant failover.

A typical cloud-native setup uses cross-region failover — say, `us-east-1` as primary and `eu-west-2` as the DR target.

### AWS Elastic Disaster Recovery (DRS)

DRS — formerly **CloudEndure Disaster Recovery** — replicates physical, virtual, or cloud-based servers continuously into AWS, so you can recover them quickly. The architecture is straightforward: an AWS Replication Agent on the source does continuous block-level replication into a low-cost staging area on AWS (cheap EC2s plus EBS), and on failover those servers are promoted to full target capacity in minutes. Typical uses include protecting critical databases (Oracle, MySQL, SQL Server), enterprise apps (SAP), and ransomware resilience.

---

## Migration

### AWS DataSync

DataSync moves large amounts of data from on-prem into AWS. It can sync into S3 (any storage class, including Glacier), EFS, and FSx for Windows. Replication tasks can be scheduled hourly, daily, or weekly, and are **incremental after the first full load**.

### The 7 Rs of migration

A useful framework for deciding what to do with each app when moving to the cloud:

- **Retire** — turn off things you don't need (often a result of re-architecting). Reduces attack surface, can save 10–20% in costs.
- **Retain** — do nothing for now. Valid for compliance, performance, unresolved dependencies, or low-value workloads (mainframes, non-x86 Unix).
- **Relocate** — move an app from on-prem to its cloud equivalent (e.g. VMware on-prem → VMware Cloud on AWS), or shift EC2s between VPCs, accounts, or regions.
- **Rehost ("lift and shift")** — re-host as-is on AWS, no cloud optimisation. Quick. Can save up to 30% on cost. Example: AWS Application Migration Service.
- **Replatform ("lift and reshape")** — move with light optimisation. Examples: DB → RDS, app → Elastic Beanstalk. Trades a small amount of change for big ops savings.
- **Repurchase ("drop and shop")** — switch to a different product, often SaaS, as part of the move. Examples: CRM → Salesforce, HR → Workday, CMS → Drupal. Expensive short-term, fast to deploy.
- **Refactor / Re-architect** — reimagine the app using cloud-native features. Monolith → microservices, EC2 → serverless. Driven by needs around scalability, performance, security, or agility.

### AWS Application Discovery Service

Discovery gathers information about on-prem data centres to plan a migration. Server utilisation data and dependency mapping are crucial inputs:

- **Agentless Discovery** — VM inventory, configuration, and performance history (CPU, memory, disk).
- **Agent-based Discovery** — system configuration, performance, running processes, and network connection details.

Results flow into AWS Migration Hub.

### AWS Application Migration Service (MGN)

MGN — the AWS evolution of CloudEndure Migration, replacing AWS Server Migration Service — is the **lift-and-shift** tool. It converts physical, virtual, and cloud-based servers to run natively on AWS, across a wide range of platforms, operating systems, and databases. Minimal downtime, reduced cost. Architecturally it looks just like DRS: an AWS Replication Agent does continuous replication into a low-cost staging area on AWS, and at cutover the staged servers become production.

### AWS Migration Evaluator

Migration Evaluator helps you build a **data-driven business case** for migration. You install the Agentless Collector on-prem, it gathers inventory and resource utilisation, and Migration Evaluator produces a Quick Insights cost report and expert guidance for a business case.

### AWS Migration Hub

Migration Hub is the **central location** for collecting server and application inventory data, planning, and tracking migrations. **Migration Hub Orchestrator** provides pre-built templates for common enterprise apps like SAP and SQL Server. It integrates status updates from both Application Migration Service (MGN) and Database Migration Service (DMS).

---

## Resilience and workflows

### AWS Fault Injection Simulator (FIS)

FIS runs **fault injection experiments** on AWS workloads — chaos engineering as a service. You stress an application by creating disruptive events (sudden CPU or memory spikes, instance failures), observe how the system responds, and improve from there. It supports EC2, ECS, EKS, and RDS, with pre-built templates for common disruptions. Experiments can be stopped automatically when they complete or when a CloudWatch alarm trips.

### AWS Step Functions

Step Functions lets you build **serverless visual workflows** to orchestrate Lambda functions — and integrates with EC2, ECS, on-prem servers, API Gateway, SQS, and more. It supports sequences, parallel branches, conditions, timeouts, error handling, and even human approval steps. Typical uses: order fulfilment, data processing, web applications, and any multi-step workflow.

---

## Specialised

### AWS Ground Station

Ground Station is a fully managed service for **satellite communications**. AWS runs a global network of ground stations near AWS regions, and you can downlink satellite data into your AWS VPC within seconds, sending it to S3 or EC2 for processing. Used for weather forecasting, surface imaging, communications, and video broadcasts.

### Amazon Pinpoint

Pinpoint is a scalable **2-way marketing communications** service — email, SMS, push, voice, and in-app messaging. You can segment audiences, personalise messages, receive replies, and scale to billions of messages a day. Use it for marketing campaigns, bulk messaging, and transactional SMS.

**Pinpoint vs SNS or SES:**

- With **SNS** and **SES** you manage each message — audience, content, and delivery schedule, one push at a time.
- With **Pinpoint** you create reusable message templates, delivery schedules, targeted segments, and full campaigns.

---

## Decision shortcuts

| Need | Service |
|------|---------|
| Cloud-hosted Windows/Linux desktops | **WorkSpaces** |
| Stream a single app to a browser | **AppStream 2.0** |
| Connect IoT devices to AWS | **IoT Core** |
| GraphQL backend with real-time sync | **AppSync** |
| Full-stack mobile/web framework | **Amplify** |
| Visual serverless design → CloudFormation | **Infrastructure Composer** |
| Test apps on real mobile devices | **Device Farm** |
| Centralised, scheduled backups | **AWS Backup** |
| Continuous server replication for DR | **Elastic Disaster Recovery** |
| Move large data from on-prem to AWS | **DataSync** |
| Discover what's running on-prem | **Application Discovery Service** |
| Lift-and-shift servers to AWS | **Application Migration Service (MGN)** |
| Build a migration business case | **Migration Evaluator** |
| Track an in-flight migration | **Migration Hub** |
| Chaos engineering / fault injection | **Fault Injection Simulator** |
| Orchestrate Lambdas into a workflow | **Step Functions** |
| Talk to satellites | **Ground Station** |
| Run marketing campaigns | **Pinpoint** |

---

## Summary

- **End-user computing**: WorkSpaces for full virtual desktops, AppStream for streaming a single app.
- **App dev**: IoT Core, AppSync, Amplify, Infrastructure Composer, Device Farm.
- **Backup & DR**: AWS Backup for routine backups, Elastic Disaster Recovery for server-level replication. Pick a DR strategy by cost-vs-recovery-time trade-off — Backup & Restore, Pilot Light, Warm Standby, or Multi-Site.
- **Migration**: the 7 Rs frame the decision; AWS Application Discovery, Application Migration Service, Migration Evaluator, and Migration Hub execute it.
- **Resilience and workflows**: Fault Injection Simulator for chaos engineering, Step Functions for orchestration.
- **Specialised**: Ground Station for satellites, Pinpoint for marketing comms.

---
