# AWS Resource Inventory

## Compute

| Resource | ID | Type | Tier | Subnet |
|---|---|---|---|---|
| EC2 Instance | i-0a1b2c3d4e5f (web-server-1) | t3.medium | Web | public-subnet-1a |
| EC2 Instance | i-0f9e8d7c6b5a (web-server-2) | t3.medium | Web | public-subnet-1b |
| EC2 Instance | i-0b2c3d4e5f6a (app-server-1) | t3.large | App | private-subnet-1a |
| EC2 Instance | i-0c3d4e5f6a7b (app-server-2) | t3.large | App | private-subnet-1b |

**AMI:** Amazon Linux 2023
**Key Pairs:** web-keypair, app-keypair

---

## Load Balancing

| Resource | Name | Type | Scheme |
|---|---|---|---|
| Application Load Balancer | web-alb | ALB | Internet-facing |

**Listener:** HTTPS:443 (certificate: ACM-managed)
**Target Group:** web-ec2-tg (targets: web-server-1, web-server-2)

---

## Database

| Resource | Identifier | Engine | Class | Multi-AZ |
|---|---|---|---|---|
| RDS Instance | main-database | PostgreSQL 15.4 | db.t3.large | Yes |
| RDS Subnet Group | db-subnet-group | — | — | private-subnet-1a, private-subnet-1b |

**Automated Backups:** Enabled (7-day retention)
**Maintenance Window:** Sunday 02:00–03:00 UTC
**Encryption at Rest:** Not Enabled ⚠️
**Deletion Protection:** Disabled ⚠️

---

## Storage

| Resource | Bucket Name | Purpose | Public Access Blocked | Encryption |
|---|---|---|---|---|
| S3 Bucket | my-app-assets | Static assets / frontend files | No ⚠️ | Not Enabled ⚠️ |
| S3 Bucket | my-app-logs | ALB and application logs | Yes | Not Enabled ⚠️ |
| S3 Bucket | my-app-backups | Database and config backups | Yes | Not Enabled ⚠️ |

---

## Networking

| Resource | ID / Name | Description |
|---|---|---|
| VPC | vpc-0x1y2z3w (app-vpc) | 10.0.0.0/16 |
| Public Subnet 1a | subnet-pub-1a | 10.0.1.0/24 — web tier |
| Public Subnet 1b | subnet-pub-1b | 10.0.2.0/24 — web tier |
| Private Subnet 1a | subnet-priv-1a | 10.0.3.0/24 — app tier |
| Private Subnet 1b | subnet-priv-1b | 10.0.4.0/24 — app tier |
| DB Subnet 1a | subnet-db-1a | 10.0.5.0/24 — data tier |
| DB Subnet 1b | subnet-db-1b | 10.0.6.0/24 — data tier |
| Internet Gateway | igw-0abc123 | Attached to app-vpc |
| NAT Gateway | nat-0def456 | In public-subnet-1a |
| Route Table | rtb-public | Associated with public subnets |
| Route Table | rtb-private | Associated with private/DB subnets |

### Security Groups

| Name | ID | Attached To | Inbound Rules |
|---|---|---|---|
| sg-alb | sg-0001 | ALB | 443 from 0.0.0.0/0, 80 from 0.0.0.0/0 |
| sg-web | sg-0002 | Web EC2 | 80/443 from sg-alb |
| sg-app | sg-0003 | App EC2 | 3000 from sg-web |
| sg-db | sg-0004 | RDS | 5432 from sg-app |

---

## Identity & Access Management

| Resource | Name | Purpose |
|---|---|---|
| IAM Role | ec2-web-role | Attached to web-server EC2s |
| IAM Role | ec2-app-role | Attached to app-server EC2s (S3/SSM permissions) |
| IAM Policy | app-s3-policy | Allows read/write to my-app-assets bucket |
| IAM Policy | ssm-policy | Allows AWS Systems Manager access |

**MFA Enforcement:** Not enabled on all IAM users ⚠️
**Root Account MFA:** Enabled ✅
**Password Policy:** Default (not hardened) ⚠️

---

## Monitoring & Logging

| Resource | Status |
|---|---|
| CloudTrail | Not Enabled ⚠️ |
| VPC Flow Logs | Not Enabled ⚠️ |
| CloudWatch Alarms | None configured ⚠️ |
| AWS Config | Not Enabled ⚠️ |
| GuardDuty | Not Enabled ⚠️ |
