---
title: "AWS Security & Compliance: A Tour of the Services"
date: 2026-05-18T16:00:00Z
draft: false
tags:
  - AWS
  - Security
  - Compliance
---

Security on AWS is a layered set of responsibilities and a long catalogue of services. AWS handles the security *of* the cloud; you handle security *in* the cloud. This post walks through that split, then through the services that protect networks, manage keys, detect threats, and prove compliance.

---

## The shared responsibility model

The foundation: AWS and the customer have clearly divided jobs.

- **AWS is responsible for security *of* the cloud** — the hardware, software, facilities, and networking that run every AWS service, plus the operations of fully managed services like S3, DynamoDB, and RDS.
- **You are responsible for security *in* the cloud** — the things you configure on top: guest OS patches on EC2, firewall and network configuration, IAM, and encrypting your application data.
- **Shared controls** — patch management, configuration management, and awareness and training span both sides.

### Example: RDS

- **AWS:** manages the underlying EC2 instance, disables SSH access to it, automates DB and OS patching, and audits the disks.
- **You:** check the security group inbound rules, create database users and permissions, choose public versus private access, enforce SSL on connections, and turn on encryption.

### Example: S3

- **AWS:** guarantees unlimited storage, encryption availability, isolation between customers, and that AWS employees can't access your data.
- **You:** configure the bucket, set the bucket policy and public-access settings, manage IAM users and roles, and turn on encryption.

---

## Network protection

### DDoS attacks

A **Distributed Denial-of-Service** attack uses many machines — botnets controlled by attackers — to overwhelm an application until normal users can't reach it. AWS provides several layers of defence.

- **AWS Shield Standard** — free, automatic, enabled for every AWS customer. Protects against common layer 3 and layer 4 attacks like SYN/UDP floods and reflection attacks.
- **AWS Shield Advanced** — optional, $3,000 per month per organization. Protects EC2, ELB, CloudFront, Global Accelerator, and Route 53. Includes 24/7 access to the AWS DDoS Response Team and protection from cost spikes caused by usage surges during an attack.
- **CloudFront and Route 53** — give availability protection through the global edge network. Combined with Shield, they mitigate attacks at the edge.
- **AWS Auto Scaling** — be ready to scale up under load.

### AWS WAF — Web Application Firewall

WAF protects web applications from **common layer 7 (HTTP) exploits**. It deploys in front of an Application Load Balancer, API Gateway, or CloudFront, and you configure it via a **Web ACL** (Web Access Control List):

- Rules can match on IP addresses, HTTP headers, HTTP body, or URI strings.
- Protects against **SQL injection** and **Cross-Site Scripting (XSS)**.
- Supports size constraints and geo-match (e.g. block specific countries).
- **Rate-based rules** count occurrences over time, useful for DDoS mitigation.

### AWS Network Firewall

Network Firewall protects an entire VPC, with **layer 3 through layer 7** inspection in any direction:

- VPC ↔ VPC traffic.
- Outbound to the internet.
- Inbound from the internet.
- Traffic to and from Direct Connect and Site-to-Site VPN.

### AWS Firewall Manager

Firewall Manager **manages security rules across all accounts of an AWS Organization** from one place. A security policy can include VPC security groups, WAF rules, AWS Shield Advanced, and AWS Network Firewall. Rules are applied to **new resources as they are created** — great for ongoing compliance across all current and future accounts.

---

## Penetration testing — what's allowed

You can run security assessments or penetration tests against your own AWS infrastructure **without prior approval** for a defined set of services: EC2, NAT Gateways, ELB, RDS, CloudFront, Aurora, API Gateway, Lambda and Lambda@Edge, Lightsail, and Elastic Beanstalk environments.

**Prohibited activities** include DNS zone walking via Route 53 hosted zones, any form of DoS or DDoS (real or simulated), port flooding, protocol flooding, and request flooding. For any other simulated event, contact `aws-security-simulated-event@amazon.com`.

---

## Encryption — at rest and in transit

Two states of data, both worth encrypting:

- **At rest** — data stored on a device: a hard disk, an RDS instance, S3, Glacier, and so on.
- **In transit (in motion)** — data being moved between locations: from on-prem to AWS, from EC2 to DynamoDB, anywhere on a network.

The services below let you encrypt both.

### AWS KMS — Key Management Service

Whenever you hear "encryption" for an AWS service, it is almost certainly **KMS** under the hood. KMS manages encryption keys for you. Some services let you opt in to encryption (EBS, S3 — though SSE-S3 is on by default, Redshift, RDS, EFS); others have it on automatically (CloudTrail logs, S3 Glacier, Storage Gateway).

**Types of KMS keys:**

- **Customer Managed Key** — you create, manage, enable, and disable it. Supports rotation policies and bring-your-own-key.
- **AWS Managed Key** — created and managed by AWS on your behalf, used by AWS services (`aws/s3`, `aws/ebs`, `aws/redshift`).
- **AWS Owned Key** — a collection of keys an AWS service owns across multiple accounts. AWS uses them to protect your resources, but you can't see them.
- **CloudHSM Keys** — keys generated from your own CloudHSM hardware, with cryptographic operations performed inside the cluster.

### CloudHSM

Where KMS is AWS-managed software, **CloudHSM is AWS-provisioned hardware** — a dedicated Hardware Security Module. You manage your own encryption keys entirely; AWS doesn't have access to them. The HSM device is tamper-resistant and FIPS 140-2 Level 3 compliant.

### AWS Certificate Manager (ACM)

ACM lets you **provision, manage, and deploy SSL/TLS certificates**. It is the path to HTTPS on AWS.

- Supports both public and private TLS certificates.
- Public certificates are **free**.
- **Automatic renewal.**
- Loads certificates onto Elastic Load Balancers, CloudFront distributions, and API Gateway APIs.

### AWS Secrets Manager

Secrets Manager is the newer service for **storing secrets** — DB passwords, API keys.

- Can **force rotation** every N days.
- Generates new secrets on rotation, using a Lambda function.
- Native integration with Amazon RDS (MySQL, PostgreSQL, Aurora).
- Secrets are encrypted using KMS.

Its sweet spot is RDS credentials, where rotation is most painful to do by hand.

---

## Compliance documentation

### AWS Artifact

Artifact is a **portal**, not really a service. It gives on-demand access to AWS compliance documentation and agreements:

- **Artifact Reports** — download AWS security and compliance documents from third-party auditors: ISO certifications, PCI, SOC reports.
- **Artifact Agreements** — review, accept, and track AWS agreements like the Business Associate Addendum (BAA) or HIPAA, for one account or across an Organization.

Use it to support internal audits or compliance reviews.

---

## Threat detection and assessment

### Amazon GuardDuty

GuardDuty is **intelligent threat detection** for your AWS account. It uses machine learning, anomaly detection, and third-party threat data. One click to enable — no software to install — with a 30-day free trial.

Input data includes:

- **CloudTrail Events Logs** — unusual API calls, unauthorized deployments (both management events and S3 data events).
- **VPC Flow Logs** — unusual internal traffic, unusual IP addresses.
- **DNS Logs** — compromised EC2 instances exfiltrating data via DNS queries.
- **Optional features** — EKS audit logs, RDS and Aurora, EBS, Lambda, S3 data events.

Findings can trigger **EventBridge rules**, which can in turn invoke Lambda or post to SNS. GuardDuty even has a dedicated finding type for **cryptocurrency-mining attacks**.

### Amazon Inspector

Inspector runs **automated security assessments** across three surfaces:

- **EC2 instances** — uses the SSM agent to check for unintended network reachability and known OS vulnerabilities.
- **Container images pushed to ECR** — assesses images as they are pushed.
- **Lambda functions** — identifies vulnerabilities in function code and package dependencies, assessing functions as they are deployed.

What it actually evaluates:

- **Package vulnerabilities** (EC2, ECR, Lambda) against a CVE database.
- **Network reachability** (EC2).
- Assigns each finding a **risk score** for prioritization.

Findings flow to Security Hub and EventBridge.

### AWS Config

Config helps with **auditing and recording compliance** of AWS resources, and tracking configuration changes over time. Configuration data can be stored in S3 for analysis with Athena.

Questions Config can answer:

- Is there unrestricted SSH access to any of my security groups?
- Do any of my buckets have public access?
- How has my ALB configuration changed over time?

You can receive SNS alerts on any change. Config is **per-region**, but can be aggregated across regions and accounts.

### Amazon Macie

Macie is a **data security and data privacy service** that uses machine learning and pattern matching to **discover sensitive data in S3** — most importantly Personally Identifiable Information (PII). It analyzes buckets, surfaces findings, and integrates with EventBridge for downstream notification.

---

## Centralisation and investigation

### AWS Security Hub

Security Hub is the **central security dashboard** across multiple AWS accounts and the place to automate security checks. It aggregates findings from a long list of sources in a common format:

- Config, GuardDuty, Inspector, Macie.
- IAM Access Analyzer.
- AWS Systems Manager.
- AWS Firewall Manager.
- AWS Health.
- AWS Partner Network solutions.

Security Hub requires AWS Config to be enabled first.

### Amazon Detective

GuardDuty, Macie, and Security Hub identify *findings* — potential issues. But isolating the root cause of one of those findings can be a complex investigation on its own. **Amazon Detective** uses machine learning and graph analysis to do exactly that: it automatically collects events from VPC Flow Logs, CloudTrail, and GuardDuty, builds a unified view, and produces visualizations with the context needed to get to the root cause.

---

## Accounts and access

### Root user privileges

The **root user** is the account owner, created when the AWS account is created. It has complete access to every AWS service and resource.

- **Lock away the root user access keys.**
- **Do not use the root account** for everyday tasks — not even administrative ones.

A handful of actions can only be performed by the root user:

- Change account settings (account name, email, root password, root access keys).
- View certain tax invoices.
- Close your AWS account.
- Restore IAM user permissions.
- Change or cancel your AWS Support plan.
- Register as a seller in the Reserved Instance Marketplace.
- Configure an S3 bucket to enable MFA Delete.
- Edit or delete an S3 bucket policy that includes an invalid VPC ID or VPC endpoint ID.
- Sign up for GovCloud.

### IAM Access Analyzer

Access Analyzer finds out **which of your resources are shared externally** — outside your defined "zone of trust." Covers S3 buckets, IAM roles, KMS keys, Lambda functions and layers, SQS queues, and Secrets Manager secrets. Define your zone of trust as an AWS account or an AWS Organization, and any access outside it shows up as a finding.

---

## AWS Abuse

If AWS resources are being used against *you* — for spam, port scanning, DoS, intrusion attempts, hosting objectionable or copyrighted content, or distributing malware — report it via the AWS Abuse form or `abuse@amazonaws.com`.

---

## Decision shortcuts

| Need | Service |
|------|---------|
| Automatic DDoS protection | **AWS Shield Standard** |
| 24/7 DDoS response and advanced protection | **AWS Shield Advanced** |
| Filter HTTP requests by rules (SQLi, XSS, rate limits) | **AWS WAF** |
| Protect a whole VPC at layers 3–7 | **AWS Network Firewall** |
| Org-wide security rules across accounts | **AWS Firewall Manager** |
| Managed encryption keys | **AWS KMS** |
| Dedicated hardware encryption, you hold the keys | **AWS CloudHSM** |
| TLS certs for HTTPS | **AWS Certificate Manager** |
| Store and rotate secrets (especially RDS creds) | **AWS Secrets Manager** |
| Download compliance reports (PCI, ISO, SOC) | **AWS Artifact** |
| Anomaly-based threat detection | **Amazon GuardDuty** |
| Vulnerability scans for EC2, ECR, Lambda | **Amazon Inspector** |
| Track resource config changes and compliance | **AWS Config** |
| Find PII in S3 | **Amazon Macie** |
| Central security dashboard | **AWS Security Hub** |
| Root-cause investigation of a finding | **Amazon Detective** |
| Find externally-shared resources | **IAM Access Analyzer** |
| Report abusive AWS resources | **AWS Abuse** |

---

## Summary

- The **shared responsibility model** is the lens for everything else: AWS secures the cloud; you secure what you put in it.
- **Network protection** is layered: Shield for DDoS, WAF for HTTP exploits, Network Firewall for the VPC, Firewall Manager to roll it across accounts.
- **Encryption** comes in four flavours: KMS for managed keys, CloudHSM for hardware-managed keys, ACM for TLS certificates, Secrets Manager for credentials.
- **Threat detection** is split between behaviour (GuardDuty), vulnerabilities (Inspector), configuration drift (Config), and sensitive data (Macie).
- **Security Hub** centralizes findings; **Detective** investigates them.
- For accounts, lock down the **root user** and use **IAM Access Analyzer** to catch anything shared outside your zone of trust.

---
