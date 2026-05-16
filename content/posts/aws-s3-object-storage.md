---
title: "AWS S3: Object Storage"
date: 2026-05-16T20:00:00Z
draft: false
tags:
  - AWS
  - S3
  - Storage
---

S3 is object storage — durable, effectively infinite, and accessed over an HTTP API. It is the default place to put blobs in AWS: images, backups, logs, static assets, anything.

---

## The core pieces

| Piece | What it is |
|-------|------------|
| **Bucket** | The top-level container for objects. Its name is globally unique, and it lives in one region. |
| **Object** | A blob plus metadata, addressed by a key. The thing you actually store. |
| **Key** | The object's "path" — e.g. `photos/2024/cat.jpg`. It is just a string; there are no real folders. |

---

## Mental model

```
S3 = giant key → blob store
Bucket (region-scoped, globally unique name)
   └── Object: key="photos/cat.jpg", value=<bytes>, metadata, version_id, tags
```

- A **bucket** is a container.
- An **object** is a key mapped to a blob, with metadata attached.
- "Folders" are just key prefixes — not real directories.

---

## Security — 3 layers

| Layer | What it is | Attached to |
|-------|------------|-------------|
| **IAM policy** | Rules on the requester | User or role |
| **Bucket policy** | Rules on the resource — supports cross-account and public access | Bucket |
| **Encryption** | Data-at-rest protection | Object |

**The access decision:** any ALLOW — from the IAM policy *or* the bucket policy — combined with no explicit DENY means access is granted.

**Encryption options:** SSE-S3 (AWS-managed keys), SSE-KMS (your KMS keys, auditable), SSE-C (you supply the keys), or client-side encryption.

---

## Versioning

- Enabled per bucket. Every overwrite creates a new version.
- A **normal delete** adds a *delete marker* on top — the object appears gone, but nothing is actually destroyed.
- **Deleting the delete marker** restores the previous version.
- **True deletion** uses `DELETE ?versionId=vX` — it targets a specific version and creates no marker.
- Once enabled, versioning can only be *suspended*, never removed.

**Why bother:** it makes delete a soft operation by default, protecting against accidental and malicious deletions.

---

## Storage classes

Storage classes trade speed and cost against each other, from fast and expensive to slow and cheap. The class is set per object.

| Class | Use |
|-------|-----|
| **Standard** | The default, for hot data |
| **Intelligent-Tiering** | Unknown access patterns — auto-moves objects between tiers |
| **Standard-IA** | Infrequent access, multi-AZ |
| **One Zone-IA** | Infrequent access in a single AZ — cheaper, less durable |
| **Glacier Instant** | Archive with instant retrieval |
| **Glacier Flexible** | Archive with minutes-to-hours retrieval |
| **Glacier Deep Archive** | The cheapest class, with 12h+ retrieval |
| **Express One Zone** | High-performance, single AZ — the opposite of Glacier |

Minimum storage durations apply: Standard-IA and One Zone-IA are 30 days, Glacier Instant and Flexible are 90 days, and Deep Archive is 180 days.

---

## Lifecycle rules

Lifecycle rules are bucket-level automation that act on objects:

- **Transitions** move objects to a cheaper class after N days. They only go one way — toward colder storage.
- **Expirations** delete objects, old versions, or incomplete multipart uploads.
- Rules can be scoped by prefix, tag, or object size.

---

## Replication (SRR / CRR)

- **SRR** is same-region replication and **CRR** is cross-region — they are mechanically identical.
- **Versioning must be enabled** on both the source and destination buckets.
- Replication is **asynchronous** and applies to *new* objects after it is enabled. Existing objects need batch replication.
- Use it for disaster recovery, compliance and data sovereignty, lower latency, log aggregation, or separating prod and dev data.

---

## Other features worth knowing

- **Static website hosting** → serve HTML, CSS, and JS directly from a bucket.
- **Pre-signed URLs** → time-limited signed links to a private object, allowing uploads and downloads without making the object public.
- **MFA Delete** → require an MFA token to delete a version.
- **Object Lock** → WORM (write-once-read-many) storage for compliance, where even the root account cannot delete the object.

---

## Running example

A **photo app** stored in S3:

- A bucket `photos-app-uploads` in `us-east-1`.
- The app uses an **IAM role** on EC2 to PUT and GET objects.
- A **bucket policy** denies any upload that is not encrypted.
- **Versioning is on**, so a user "delete" only hides the object — an admin can recover it.
- **Lifecycle rules:** Standard → Standard-IA after 30 days → Glacier after 180 days, with old versions expiring after a year.
- **CRR** to `us-west-2` for disaster recovery.
- **Pre-signed URLs** let users download their photos directly from S3 without routing through EC2.

---

## Summary

- S3 is a giant key → blob store: per-region buckets holding objects addressed by a key.
- Security comes in three layers — IAM policies, bucket policies, and encryption.
- Versioning, lifecycle rules, and replication layer on top to protect data, control cost, and meet durability needs.
- Reach for S3 whenever you need durable storage for blobs — and keep EBS, Instance Store, and EFS for running the app itself.

---
