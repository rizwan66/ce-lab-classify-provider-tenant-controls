# AWS Shared Responsibility Assessment Report

**Prepared by:** Cloud Security Engineer
**Date:** 2026-03-23
**Environment:** Three-Tier Web Application (Development/Production)
**AWS Region:** us-east-1

---

## Executive Summary

This report documents the security responsibilities for a three-tier web application running on AWS, consisting of an Application Load Balancer, EC2-based web and application tiers, an RDS PostgreSQL database, and S3 storage.

Using the AWS Shared Responsibility Model as a framework, we inventoried **41 security controls** across seven domains and identified **16 gaps** — 4 Critical, 5 High, 4 Medium, and 3 Low priority. Immediate action is required on 4 critical gaps, particularly the absence of CloudTrail logging, unencrypted S3 buckets, and disabled GuardDuty threat detection. A full remediation roadmap with effort estimates is provided.

**Key finding:** Because this architecture uses EC2 (IaaS), the customer bears significantly more security responsibility than if the same workload were hosted on a fully managed PaaS or serverless platform.

---

## Architecture Overview

```
Internet
    │
    ▼
┌─────────────────────────────────────────┐
│   Application Load Balancer (ALB)       │  ← PaaS: AWS manages infrastructure
│   HTTPS:443 — Internet-facing           │
└─────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────┐
│   Web Tier — Public Subnets             │  ← IaaS: Customer manages OS + app
│   EC2: web-server-1, web-server-2       │
│   (Amazon Linux 2023, t3.medium)        │
└─────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────┐
│   App Tier — Private Subnets            │  ← IaaS: Customer manages OS + app
│   EC2: app-server-1, app-server-2       │
│   (Node.js runtime, t3.large)           │
└─────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────┐
│   Data Tier — DB Subnets (Private)      │  ← PaaS: AWS manages OS + engine
│   RDS PostgreSQL 15.4, Multi-AZ         │
└─────────────────────────────────────────┘

Supporting Services:
  S3 (3 buckets) — Assets, Logs, Backups
  IAM — Roles and policies for EC2 instances
  VPC — Custom network with public/private/DB subnets
```

---

## Service Model Classification

| Service | Model | Customer Responsibility Level | Key Customer Obligations |
|---|---|---|---|
| EC2 (web tier) | IaaS | **High** | Guest OS patching, hardening, application security, key management |
| EC2 (app tier) | IaaS | **High** | Guest OS patching, Node.js security, dependency scanning, secrets management |
| RDS PostgreSQL | PaaS | **Medium** | Enable encryption, manage DB users/privileges, schedule maintenance, enforce TLS |
| S3 | PaaS | **Medium** | Enable encryption, block public access, bucket policies, versioning, access logging |
| ALB | PaaS | **Low** | Configure listeners, TLS policy, HTTP→HTTPS redirect, target group health checks |
| VPC / Networking | IaaS | **High** | Subnet design, routing, security group rules, NACLs, flow logs |
| IAM | Platform | **High** | All identity and access controls — AWS manages the engine only |

**Key insight:** EC2-based workloads (IaaS) carry the highest customer responsibility. Migrating the application tier to AWS Lambda or containers on ECS Fargate would shift OS patching responsibility to AWS, reducing the attack surface the customer must manage.

---

## Shared Responsibility Model — Visual Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                     AWS RESPONSIBILITY                          │
│                                                                 │
│  Physical security · Hardware lifecycle · Hypervisor isolation  │
│  Networking backbone · AZ/Region infrastructure                 │
│  RDS / ALB host OS · Managed service patching                  │
│  S3 durability · IAM policy engine · Shield Standard DDoS      │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   CUSTOMER RESPONSIBILITY                       │
│  (All items below are customer-owned in this architecture)      │
│                                                                 │
│  EC2 guest OS patching    │  IAM policies & MFA enforcement     │
│  Application security     │  S3 encryption & access control     │
│  RDS encryption at rest   │  Security group rules               │
│  Database user privileges │  VPC design & flow logs             │
│  CloudTrail logging       │  GuardDuty & monitoring             │
│  TLS enforcement (app)    │  Incident response plan             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Responsibility Matrix Summary

| Domain | Total Controls | AWS Managed | Customer Managed | Shared | Gaps Found |
|---|---|---|---|---|---|
| Physical & Infrastructure | 5 | 4 | 0 | 1 | 0 |
| Compute (EC2) | 8 | 2 | 6 | 0 | 5 |
| Network Security | 6 | 1 | 3 | 2 | 3 |
| Database (RDS) | 7 | 2 | 4 | 1 | 4 |
| Data Security (S3) | 6 | 1 | 5 | 0 | 4 |
| Identity & Access (IAM) | 5 | 1 | 4 | 0 | 3 |
| Monitoring & Logging | 4 | 0 | 4 | 0 | 4 |
| **Total** | **41** | **11** | **26** | **4** | **16** |

Full control-by-control breakdown is documented in `responsibility-matrix.md`.

---

## Gap Analysis Summary

| ID | Gap | Priority | Effort |
|---|---|---|---|
| GAP-01 | S3 buckets not encrypted at rest | Critical | 1 hour |
| GAP-02 | S3 public access not blocked (my-app-assets) | Critical | 30 min |
| GAP-03 | CloudTrail not enabled | Critical | 2 hours |
| GAP-04 | GuardDuty not enabled | Critical | 1 hour |
| GAP-05 | RDS encryption not enabled | High | 1 week |
| GAP-06 | EC2 guest OS patching not automated | High | 3 days |
| GAP-07 | IMDSv2 not enforced (SSRF exposure) | High | 1 hour |
| GAP-08 | MFA not enforced for IAM users | High | 1 day |
| GAP-09 | VPC Flow Logs not enabled | High | 2 hours |
| GAP-10 | S3 versioning not enabled | Medium | 1 hour |
| GAP-11 | Security groups need formal audit | Medium | 2 days |
| GAP-12 | RDS TLS not enforced in application | Medium | 1 day |
| GAP-13 | Database least-privilege not reviewed | Medium | 1 day |
| GAP-14 | AWS Config not enabled | Low | 1 day |
| GAP-15 | CloudWatch alarms not configured | Low | 1 day |
| GAP-16 | S3 server access logging not enabled | Low | 1 hour |

Full gap details and remediation steps are documented in `gap-analysis.md`.

---

## Recommendations

### Immediate (This Week)

1. **Enable CloudTrail** — With zero audit logging, the organization has no forensic capability and fails every major compliance framework. This must be the first action taken. Estimated time: 2 hours.

2. **Encrypt all S3 buckets and block public access** — These are one-line API calls that eliminate two of the most common causes of S3 data breaches. Estimated time: 1-2 hours total.

3. **Enable GuardDuty** — Provides automated, ML-based threat detection across the account for a low monthly cost (~$3–5/month at this scale). Any active compromise will be flagged immediately. Estimated time: 1 hour.

4. **Enforce IMDSv2 on all EC2 instances** — Prevents the SSRF-based metadata credential theft attack pattern (see Capital One breach bonus analysis). A one-command fix per instance. Estimated time: 1 hour.

### Short Term (Within 2 Weeks)

5. **Automate EC2 OS patching with SSM Patch Manager** — Unpatched guest OS is the most common attack vector for EC2 workloads. Estimated time: 3 days setup.

6. **Enforce MFA on all IAM users** — A compromised console credential without MFA gives full account access. Estimated time: 1 day.

7. **Enable VPC Flow Logs** — Required for any incident response capability. Estimated time: 2 hours.

8. **Begin RDS encryption migration** — Requires a blue/green cutover. Plan, test, and execute carefully to avoid data loss. Estimated time: 1 week including validation.

### Medium Term (Within 30 Days)

9. **Review and tighten all security groups** — Prioritize removing any 0.0.0.0/0 rules beyond port 443 on sg-alb.

10. **Enforce TLS for RDS connections and rotate to least-privilege DB user** — Eliminates plaintext database traffic and reduces SQL injection blast radius.

---

## Compliance Implications

The 4 Critical gaps (no CloudTrail, no S3 encryption, no public access controls, no GuardDuty) would result in **immediate failures** under:

- **PCI DSS** — Requirements 2, 3, 7, 10 (logging, encryption, access control)
- **SOC 2 Type II** — CC6, CC7 (logical access, monitoring)
- **HIPAA** (if PII/PHI data is present) — 164.312(a)(2)(iv), 164.312(b) (encryption, audit controls)
- **AWS Foundational Security Best Practices** — Multiple controls in the S3, CloudTrail, and IAM standards

---

## Conclusion

The AWS Shared Responsibility Model makes clear that while AWS secures the cloud infrastructure, the security **in** the cloud — including the OS, applications, data, identity controls, and logging — is the customer's responsibility. This architecture, built primarily on IaaS (EC2) services, places the majority of security controls in customer hands.

Of the 41 controls analyzed, 26 are customer-managed. We found 16 gaps across those controls, with 4 requiring immediate remediation. Implementing all recommendations within the proposed 30-day roadmap will establish a strong security baseline and bring the environment into compliance with common frameworks.

A key architectural recommendation beyond this assessment: evaluate migrating the app tier from EC2 to a managed compute service (ECS Fargate or Lambda) to shift OS patching responsibility to AWS and reduce the customer-managed attack surface.

---

## Document Index

| File | Description |
|---|---|
| `security-inventory.md` | Complete AWS resource inventory |
| `responsibility-matrix.md` | Full 41-control responsibility matrix with service model classification |
| `gap-analysis.md` | Detailed gap findings with remediation steps and effort estimates |
| `assessment-report.md` | This document — executive summary and recommendations |
| `bonus-breach-analysis.md` | Capital One breach analysis (bonus) |
