---
title: "AWS IAM: Identity & Access Management"
date: 2026-05-16T10:00:00Z
draft: false
tags:
  - AWS
  - IAM
  - Security
---

IAM is the part of AWS that controls **who can do what**. Every API call, every console click, and every request a service makes is checked against IAM before it is allowed to proceed.

---

## The 4 building blocks

IAM is built from four pieces. Understanding what each one is for makes the rest of the service easy to reason about.

| Block | What it is | Used for |
|--------|------------|----------|
| **User** | Long-lived credentials | Humans |
| **Group** | A bundle of users | Sharing permissions |
| **Role** | A temporary, assumed identity | Services, federated users, cross-account access |
| **Policy** | JSON rules | Attached to a user, group, role, or resource |

---

## Mental model

```
Principal (user/role) ──has──> Policy ──grants──> Permissions on resources
```

It helps to keep these one-line definitions in mind:

- **User** — *who you are.* Long-lived keys tied to a person.
- **Role** — *what you become.* Temporary STS credentials.
- **Group** — a bundle of users.
- **Policy** — the rules.

---

## How it actually works — a running example

The cleanest way to understand IAM is to follow a single example all the way through.

**Setup:** Dave is a developer who needs to upload photos to an S3 bucket.

1. An admin creates an IAM **user** `dave` — this gives him a password and access keys.
2. The admin creates a **group** `Developers` and adds Dave to it.
3. The admin writes a **policy**:

   ```json
   {
     "Effect": "Allow",
     "Action": ["s3:PutObject", "s3:GetObject"],
     "Resource": "arn:aws:s3:::photos/*"
   }
   ```

4. The policy is attached to the **group**. Dave inherits it through his group membership.

When Dave runs `aws s3 cp pic.jpg s3://photos/`, AWS asks a single question: *does any policy on Dave — or on his groups — allow `s3:PutObject` on that resource?* The answer is yes, so the upload is allowed.

### Now move the same workload to a server

Dave's app now runs on EC2 and needs the exact same S3 access. Using Dave's keys would mean copying credentials onto the server — so instead:

1. The admin creates a **role** `PhotoAppRole`.
2. The admin attaches the same S3 policy to the role (an identity-based policy, this time on a role).
3. The admin writes a **trust policy** on the role that says *"the EC2 service is allowed to assume me."*
4. The role is attached to the EC2 instance.

The app on EC2 automatically receives **temporary STS credentials**, calls S3, and the same permission check runs. No keys sit on disk, and the credentials rotate automatically. This is exactly why **roles beat users for applications**.

---

## Policy types — what they attach to

Policies come in three flavours, distinguished by what they are attached to:

- **Identity-based** — attached to a user, group, or role. Says *"this principal can do X."*
- **Resource-based** — attached to the resource itself, such as an S3 bucket policy. Says *"anyone matching X can access me."*
- **Trust policy** — attached only to a role. Defines *who is allowed to assume it.*

For an S3 bucket, the access decision uses **both** the identity policies and the bucket policy together.

---

## Evaluation rule

When IAM evaluates a request, it follows a strict precedence:

```
Explicit DENY  >  Explicit ALLOW  >  Default DENY
```

- One **explicit DENY** anywhere wins — it cannot be overridden.
- Otherwise, any **explicit ALLOW** grants access.
- If no rule matches at all, the request is **denied by default**.

---

## Common patterns

Most real-world setups fall into one of these shapes:

- **Human** → a user in a group, with the policy on the group.
- **App on EC2/Lambda** → a role, with the policy on the role.
- **Cross-account** → a role in account A, whose trust policy lets account B assume it.
- **SSO/SAML** → a federated identity assumes a role.

---

## Key principles

- Apply **least privilege** — grant only the permissions actually needed.
- Prefer **roles over users**, even for humans (via SSO).
- **Never embed credentials** — use roles instead.
- Enforce **MFA** on humans, especially the root account.
- Treat the **root account as emergency-only**.

---

## Extras (refinements)

Once the basics are in place, three features help you scale IAM safely:

- **Permission boundaries** — set the maximum permissions a user or role can ever have.
- **SCPs (Service Control Policies)** — org-wide guardrails applied across multiple accounts.
- **ABAC (Attribute-Based Access Control)** — permissions decided by tags rather than fixed policies.

---

## Summary

- IAM answers one question: *who can do what?*
- **Users** are long-lived identities, **roles** are temporary ones, **groups** bundle users, and **policies** are the rules.
- Permissions are granted by attaching policies to principals or resources.
- Evaluation is simple: an explicit deny always wins, otherwise an allow grants access, and anything unmatched is denied.
- For applications, always reach for roles — temporary credentials that rotate automatically beat keys on disk every time.

---
