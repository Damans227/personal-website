---
title: "AWS Deployments: Managing Infrastructure at Scale"
date: 2026-05-17T18:00:00Z
draft: false
tags:
  - AWS
  - DevOps
  - Infrastructure
---

Once you understand the building blocks of AWS, the next question is practical: how do you get an app *onto* AWS, and how do you manage it afterward? Five services cover this, and they group neatly by the job they do.

---

## The mental map

```
┌─────────────────────────────────────────────────────┐
│  DEFINE infrastructure as code                      │
│  • CloudFormation  (YAML/JSON templates)            │
│  • CDK             (real code → compiles to CFN)    │
├─────────────────────────────────────────────────────┤
│  DEPLOY applications                                │
│  • Elastic Beanstalk  (PaaS, all-in-one)            │
│  • CodeDeploy         (push code to existing fleet) │
├─────────────────────────────────────────────────────┤
│  MANAGE running infrastructure                      │
│  • Systems Manager (SSM)                            │
└─────────────────────────────────────────────────────┘
```

---

## CloudFormation

CloudFormation is **declarative AWS infrastructure as code**. You write a YAML or JSON template describing *what* you want, and CloudFormation creates it — in the right order, with the exact config.

```
Template:                    CloudFormation:
  - 1 security group          1. Create SG
  - 2 EC2 instances           2. Create EC2s (using that SG)
  - 1 S3 bucket               3. Create bucket
  - 1 ELB                     4. Create ELB (in front of EC2s)
```

**Why it matters:**

- Infrastructure becomes repeatable and version-controlled.
- It rolls back automatically if anything fails.
- You can replicate a whole stack across regions or accounts.
- A one-click delete tears everything down — no orphaned resources.

**One-liner:** describe the infrastructure, and AWS builds it.

---

## CDK — Cloud Development Kit

The CDK lets you **write infrastructure in real code** — Python, TypeScript, Java, .NET. The CDK compiles that code into a CloudFormation template, which CloudFormation then deploys.

```
Your code (Python/TS) ──► CDK CLI ──► CloudFormation Template ──► AWS
```

**Why use it instead of raw CloudFormation:**

- Loops, conditionals, functions, and classes.
- Reusable components.
- Strong typing and IDE autocomplete.
- You can mix infrastructure and app code — great for Lambda and ECS.

**One-liner:** programmatic infrastructure that becomes CloudFormation under the hood.

---

## Elastic Beanstalk

Beanstalk is a **developer-friendly PaaS**. You upload your app code, and Beanstalk provisions and manages everything underneath — EC2, ASG, ELB, RDS.

- **You handle:** the application code.
- **Beanstalk handles:** the OS, instance config, deployment strategy, capacity, load balancing, auto-scaling, and health monitoring.

It offers three architecture models:

- **Single instance** — for dev and test.
- **Load balancer + ASG** — for production web apps.
- **ASG only** — for non-web workers in production.

Beanstalk itself is free; you pay only for the EC2, RDS, and other resources underneath.

**One-liner:** "Just deploy my code, AWS." It is a managed wrapper around EC2 + ASG + ELB for developers who don't want to wire it all up themselves.

---

## CodeDeploy

CodeDeploy handles **automated app deployment to existing servers** — EC2 or on-prem.

```
v1 EC2s  ──CodeDeploy──►  v2 EC2s
```

Unlike Beanstalk, which provisions the infrastructure, CodeDeploy assumes the **servers already exist** — it just pushes new code to them.

**Key points:**

- Works on EC2 *and* on-prem servers, so it suits hybrid setups.
- Servers need the **CodeDeploy Agent** installed.
- Supports rolling, blue/green, and canary deployments.
- Pairs with CodePipeline for full CI/CD.

**One-liner:** push application code onto a fleet of existing servers, safely and automatically.

---

## Systems Manager (SSM)

SSM lets you **manage your fleet at scale** — EC2 and on-prem, in a hybrid setup. It is a suite of 10+ tools; the most important ones are:

- **Patch Manager** — patches your fleet automatically.
- **Run Command** — executes commands across many servers at once.
- **Parameter Store** — stores config and secrets, such as DB passwords and API keys.
- **Session Manager** — gives you a shell into an instance without opening port 22 or managing keys.

It works across Linux, Windows, macOS, and even Raspberry Pi.

**One-liner:** day-2 ops for your fleet — patching, command execution, config storage, and secure access.

---

## How they fit together

```
   Develop                Define infra          Deploy app           Operate
  ─────────              ───────────         ────────────         ─────────
   Your code   ──►   CloudFormation/CDK  ──►  Beanstalk/CodeDeploy  ──►  SSM
                     (creates EC2s,           (puts your code         (patches,
                      ELB, RDS, etc.)          on those EC2s)          runs cmds,
                                                                       stores config)
```

---

## Quick decision shortcuts

| Need | Service |
|------|---------|
| Define AWS infra in YAML | **CloudFormation** |
| Define AWS infra in Python/TS | **CDK** |
| "Just run my app, handle everything" | **Elastic Beanstalk** |
| Deploy code to existing EC2 / on-prem | **CodeDeploy** |
| Patch, SSH, run commands at scale | **SSM** |
| Store secrets / config values | **SSM Parameter Store** |

---

## Comparisons that tend to confuse people

**Beanstalk vs CodeDeploy:**

- Beanstalk provisions infra *and* deploys the app — a full PaaS.
- CodeDeploy deploys the app onto *pre-existing* infra.

**CloudFormation vs Beanstalk:**

- CloudFormation is infra-as-code for any resources, in any shape.
- Beanstalk is an opinionated stack for web apps — and it uses CloudFormation under the hood.

**CDK vs Terraform:**

- CDK is AWS-only and compiles to CloudFormation.
- Terraform is multi-cloud with its own separate state.

**SSM Parameter Store vs Secrets Manager:**

- Parameter Store is free and simple — config plus light secrets.
- Secrets Manager is paid, with auto-rotation, and is designed for DB credentials.

---

## Summary

- **CloudFormation and CDK build** the infrastructure.
- **Beanstalk and CodeDeploy deploy** the app.
- **SSM manages** the fleet day-to-day.
- Three jobs in the infrastructure-as-code and deployment lifecycle: define, deploy, operate.

---
