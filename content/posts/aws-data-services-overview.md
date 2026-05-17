---
title: "AWS Data Services: A Mental Map"
date: 2026-05-17T10:00:00Z
draft: false
tags:
  - AWS
  - Database
  - Analytics
---

AWS has a lot of data services, and the names alone don't tell you what each one is for. This post is a mental map — how the database, analytics, and data-movement services fit together, and how to pick between them.

---

## The mental map

```
┌─────────────────────────────────────────────────────────────┐
│  DATABASES (operational, low latency)                       │
├─────────────────────────────────────────────────────────────┤
│  RDS                relational SQL                          │
│  DynamoDB           NoSQL key-value, serverless              │
│  ElastiCache        in-memory cache (Redis/Memcached)        │
│  DocumentDB         MongoDB-compatible document DB           │
│  Neptune            graph DB (nodes + edges)                 │
│  Timestream         time-series DB                           │
│  Managed Blockchain Hyperledger/Ethereum                     │
├─────────────────────────────────────────────────────────────┤
│  ANALYTICS (batch, big data, querying)                      │
├─────────────────────────────────────────────────────────────┤
│  Redshift           data warehouse (columnar SQL at scale)   │
│  EMR                managed Hadoop/Spark cluster             │
│  Athena             SQL queries directly on S3               │
│  QuickSight         BI dashboards/visualization              │
├─────────────────────────────────────────────────────────────┤
│  DATA MOVEMENT                                              │
├─────────────────────────────────────────────────────────────┤
│  Glue               managed ETL (extract/transform/load)     │
│  DMS                Database Migration Service               │
└─────────────────────────────────────────────────────────────┘
```

---

## Databases

These are operational, low-latency stores — the databases an application reads and writes in real time.

### DynamoDB — NoSQL key-value, serverless

- Fully managed and serverless — no instances, no patching, no sizing.
- A key-value and document store.
- Single-digit millisecond latency at any scale.
- Auto-scales reads and writes; pay per request or with provisioned capacity.
- Multi-region with **Global Tables** — active-active writes across regions.

Use it for high-scale apps, gaming leaderboards, session storage, IoT — anything where the schema is flexible and you want zero ops.

**vs RDS:** RDS is relational SQL with joins and transactions. DynamoDB is key-based lookup at massive scale, with no joins.

**Global Tables** are multi-region and multi-master — you can write in any region. Replication is asynchronous with last-writer-wins conflict resolution, which suits globally distributed apps that need low-latency reads and writes everywhere.

### ElastiCache — in-memory cache

- Managed Redis or Memcached.
- Microsecond latency, since it is RAM-based.
- Not durable — it is a cache, not a primary store.
- Use it to cache DB query results, store sessions, build leaderboards, or do pub/sub (Redis).

It sits in front of RDS or DynamoDB to absorb hot reads:

```
EC2 ──> ElastiCache (hit?) ──yes──> return cached
                       └──no───> RDS ──> cache result, return
```

### DocumentDB — MongoDB-compatible

- A managed document store, compatible with the MongoDB API.
- For apps already using MongoDB that want managed AWS hosting.
- Storage auto-scales; designed for JSON-like documents.

### Neptune — graph database

- For data that is all about relationships — social graphs, fraud detection, recommendation engines, knowledge graphs.
- Supports Gremlin (property graph) and SPARQL (RDF) query languages.
- Models data as nodes, edges, and properties.

### Timestream — time-series database

- Optimized for time-stamped data — IoT sensors, app metrics, financial ticks.
- Auto-tiers recent data to memory and older data to magnetic storage.
- Has built-in time-series functions such as interpolation and smoothing.
- Much cheaper than storing time-series data in RDS.

### Managed Blockchain

- Managed Hyperledger Fabric or Ethereum networks.
- Niche — supply chain, finance, asset tracking. Skim it and move on unless your domain needs it.

---

## Analytics

These services are about batch processing, big data, and querying large datasets — not real-time operational reads and writes.

### Redshift — data warehouse

- A columnar, massively parallel (MPP) SQL database.
- Built for OLAP — analytical queries on huge datasets — not OLTP.
- Handles petabyte scale, complex queries, and joins across billions of rows.
- A Redshift Serverless option is available.

**vs RDS:** RDS is row-based and optimized for transactions. Redshift is column-based and optimized for "scan a billion rows and aggregate."

**vs Athena:** Redshift loads data into a cluster and is fast on repeated queries. Athena queries S3 directly with no cluster.

### EMR — Elastic MapReduce

- A managed Hadoop, Spark, Hive, or Presto cluster.
- For big data processing (TB to PB), ML pipelines, and custom data transformation.
- You manage the cluster sizing and autoscaling; AWS handles the install and config.
- Use it when you need code-level data processing power, not just SQL.

### Athena — query S3 with SQL

- Serverless — no cluster to manage.
- Run SQL directly against files in S3 (CSV, JSON, Parquet, ORC).
- Pay per query, by the TB scanned.
- Backed by Presto under the hood.

Use it for ad-hoc analysis of S3 data, log analysis, and occasional queries. It pairs naturally with Glue, which catalogs the S3 data.

### QuickSight — BI dashboards

- AWS's competitor to Tableau and Power BI.
- Connects to Redshift, RDS, Athena, S3, and more.
- Drag-and-drop visualizations, dashboards, and ML insights.
- For non-technical users who want to explore data.

---

## Data movement

### Glue — managed ETL

- Serverless ETL — extract, transform, load.
- Crawls data sources and builds a **Data Catalog** of table definitions.
- Auto-generates Spark/Python ETL jobs.
- Outputs to S3, Redshift, RDS, and others.

A common pattern:

```
Raw S3 data → Glue Crawler → Data Catalog
                                 │
                                 ▼
                            Athena / Redshift Spectrum
                            can now query S3 like a DB
```

Glue is, quite literally, the glue between raw storage and the analytics services.

### DMS — Database Migration Service

DMS moves databases into AWS, or between databases. It supports both homogeneous migrations (Oracle → Oracle) and heterogeneous ones (Oracle → Aurora).

It has two parts:

- **DMS** moves the data and can do continuous replication, so the source stays live.
- **SCT** (Schema Conversion Tool) converts the schema and stored procedures for heterogeneous moves.

Typical use cases:

- On-prem MySQL → RDS MySQL (a lift-and-shift).
- Oracle → Aurora PostgreSQL (re-platforming to save on licensing).
- Continuous replication for a zero-downtime migration.
- Populating a data lake (DB → S3).

The mental model:

```
SOURCE DB ──> DMS replication instance ──> TARGET DB
(on-prem,     (runs in your VPC,           (RDS, Aurora,
 RDS, etc.)    handles transfer)            S3, Redshift, etc.)
```

---

## How to pick — decision shortcuts

| Need | Service |
|------|---------|
| SQL, transactions, joins | RDS / Aurora |
| Massive scale, key lookup, no joins | DynamoDB |
| Fast in-memory cache | ElastiCache |
| MongoDB workload | DocumentDB |
| Graph (social, fraud) | Neptune |
| Time-series (IoT, metrics) | Timestream |
| Huge analytical queries | Redshift |
| Big data processing (Spark) | EMR |
| Query files in S3 with SQL | Athena |
| Dashboards | QuickSight |
| Build a data catalog / ETL | Glue |
| Migrate a DB into AWS | DMS |

---

## Summary

- **Databases** are operational and low-latency: RDS for relational SQL, DynamoDB for key-value at scale, ElastiCache for caching, and the rest for specialized workloads.
- **Analytics** services are for big-data querying: Redshift for warehousing, EMR for code-level processing, Athena for SQL on S3, QuickSight for dashboards.
- **Data movement** ties it together: Glue for ETL and cataloging, DMS for migrations.
- When in doubt, work backward from the shape of your data and your query pattern — that points straight at the service.

---
