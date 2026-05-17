---
title: "AWS RDS: Relational Database Service"
date: 2026-05-16T22:00:00Z
draft: false
tags:
  - AWS
  - RDS
  - Database
---

RDS is a managed SQL database. AWS runs the database server — patching, backups, failover — and you just use it. It is where the structured data of an application lives.

---

## What RDS is

- A **managed relational database**: you pick an engine, and AWS does the operations.
- Supported engines: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and **Aurora** — AWS's own MySQL/Postgres-compatible engine.
- You connect with standard SQL drivers (JDBC/ODBC) over a hostname and port.
- Under the hood it runs on an EC2-like instance, but one you never manage.

---

## Where it fits

```
ELB ──> EC2s (ASG) ──> RDS  ←── structured data (users, orders, metadata)
                  └──> S3   ←── blobs (photos, files, backups)
```

There are two storage layers doing two jobs:

- **S3** holds blobs, files, and large objects.
- **RDS** holds rows, relations, queries, and transactions.

---

## Why managed, rather than running MySQL yourself on EC2

With RDS, AWS handles:

- Automated backups, with point-in-time recovery.
- Patching of both the OS and the database engine.
- Monitoring and metrics.
- Multi-AZ failover.
- Read replicas.
- Storage scaling.

The trade-off: you can't SSH into the host. You give up some control in exchange for far less operational work.

---

## Key features

**Multi-AZ (high availability):**

- A synchronous standby copy in another Availability Zone.
- Automatic failover if the primary fails — roughly 60 seconds.
- The standby is *not* for reads — it is a hot spare.

**Read Replicas (scaling reads):**

- Asynchronous copies that are readable.
- Up to 15 with Aurora, or 5 with the other engines.
- Can be in the same region, cross-AZ, or cross-region.
- Use them to offload reporting and analytics queries from the primary.

**Storage:**

- EBS-backed (`gp3` or `io1`).
- Optional storage auto-scaling.
- Encryption at rest via KMS.

**Backups:**

- Automated backups — a daily snapshot plus transaction logs, giving point-in-time restore up to 35 days.
- Manual snapshots — kept until you delete them.

---

## Running example

A **photo app** needs a database for users, albums, and comments:

1. Create an RDS instance: PostgreSQL, `db.t3.medium`, Multi-AZ enabled, in a private subnet.
2. The security group allows port 5432 only from the EC2's security group — not from the world.
3. The EC2 app connects via the RDS endpoint hostname.
4. Credentials live in AWS Secrets Manager (rotated automatically) and are fetched by the IAM role on EC2.
5. Automated backups are set to a 7-day retention.
6. A read replica is added for the analytics dashboard, so reports don't slow down user queries.
7. If the primary AZ dies, Multi-AZ failover kicks in, the app reconnects, and downtime is about 60 seconds.

---

## Aurora — worth knowing separately

- AWS's cloud-native database, MySQL/Postgres-compatible.
- 5x MySQL or 3x Postgres performance, per AWS.
- Storage auto-scales up to 128 TB, replicated six times across three AZs.
- Up to 15 read replicas, with much faster replication.
- **Aurora Serverless** auto-scales capacity, so you pay per use.
- More expensive than plain RDS, but considerably more capable.

---

## Security

- Keep it **in a VPC**, in a private subnet — never publicly exposed.
- The **security group** is the firewall — allow traffic only from the app's security group.
- **IAM authentication** is optional — authenticate with IAM credentials instead of a password.
- **Encryption at rest** (KMS) must be set at creation time.
- **Encryption in transit** uses TLS.

---

## RDS vs S3 — when to use which

| Need | Use |
|------|-----|
| User accounts, orders, structured data | RDS |
| SQL queries, joins, transactions | RDS |
| Photos, videos, files, blobs | S3 |
| Logs, backups, data lake | S3 |
| Anything large and unstructured (> 100 KB) | S3 |

A common pattern: RDS stores the *metadata* about a blob, and S3 stores the *blob* itself. For example, a `users` table has an `avatar_url` column pointing to an S3 key.

---

## Key principles

- Put RDS in a **private subnet** — never public.
- Use **Multi-AZ** for production high availability, and **read replicas** for read scaling.
- Don't store blobs in RDS — put them in S3 and store the URL in RDS.
- Use **Secrets Manager** for database credentials, not environment variables.
- Backups are automatic — but test your restore process.

---

## Summary

- RDS is a managed SQL database that slots in behind your EC2s to hold structured data.
- AWS handles the operational heavy lifting: backups, patching, failover, and replicas.
- The division of labour is simple: **S3 holds blobs, RDS holds rows.**

---
