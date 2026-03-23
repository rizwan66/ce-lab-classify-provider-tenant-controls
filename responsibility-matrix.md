# Security Responsibility Matrix

## Service Model Classification

| Service | Model | Justification |
|---|---|---|
| EC2 (web & app tier) | IaaS | AWS provides virtualized compute; customer manages guest OS, runtime, application, and data |
| RDS PostgreSQL | PaaS | AWS manages host OS and database engine patching; customer manages schema, users, config, and data |
| S3 | PaaS / Object Storage | AWS manages the storage infrastructure and durability; customer controls access policies, encryption settings, and data |
| Application Load Balancer (ALB) | PaaS | AWS manages the infrastructure; customer configures listeners, rules, target groups, and TLS certificates |
| VPC / Subnets / Route Tables / IGW | IaaS | AWS provides the network infrastructure; customer designs topology, routing, and segmentation |
| IAM | SaaS / Platform Control Plane | AWS provides the IAM engine; customer is fully responsible for identities, policies, roles, and least-privilege enforcement |
| AWS KMS | PaaS | AWS manages key storage hardware; customer manages key policies, rotation, and usage |
| AWS Systems Manager | PaaS | AWS manages the SSM platform; customer manages agents, patch baselines, and automation |
| AWS Shield Standard | SaaS | AWS provides automatic DDoS protection with no customer action required |
| AWS CloudTrail | PaaS | AWS provides the logging infrastructure; customer enables trails and sets retention |

---

## Responsibility Matrix

### 1. Physical & Infrastructure Security

| # | Control | Service | AWS Responsibility | Customer Responsibility | Status |
|---|---|---|---|---|---|
| 1 | Physical data center security | All | ✅ Guards, locks, surveillance, CCTV, biometrics | ❌ None | ✅ AWS Managed |
| 2 | Hardware lifecycle management | All | ✅ Procurement, maintenance, decommissioning | ❌ None | ✅ AWS Managed |
| 3 | Network infrastructure security | All | ✅ Core AWS network devices, backbone, DDoS baseline | ❌ None | ✅ AWS Managed |
| 4 | Hypervisor security & isolation | EC2 | ✅ Hypervisor patching, VM isolation enforcement | ❌ None | ✅ AWS Managed |
| 5 | Availability Zone & region redundancy | All | ✅ Facility-level power, cooling, redundancy | ✅ Architect for Multi-AZ (customer choice) | ✅ RDS Multi-AZ enabled |

---

### 2. Compute Security (EC2)

| # | Control | Service | AWS Responsibility | Customer Responsibility | Status |
|---|---|---|---|---|---|
| 6 | Host OS (underlying hardware host) | EC2 | ✅ Patched and secured by AWS | ❌ None | ✅ AWS Managed |
| 7 | Guest OS patching (Amazon Linux 2023) | EC2 | ❌ None | ✅ Apply OS security updates regularly | ⚠️ TODO |
| 8 | Guest OS hardening | EC2 | ❌ None | ✅ CIS benchmark, disable unused services | ⚠️ TODO |
| 9 | Antivirus / EDR on instances | EC2 | ❌ None | ✅ Install and manage endpoint protection | ❌ Not Implemented |
| 10 | Application code security | EC2 | ❌ None | ✅ Secure coding, OWASP compliance, dependency scanning | ⚠️ TODO |
| 11 | IAM instance role configuration | EC2 | ✅ Role attachment infrastructure | ✅ Define least-privilege policies for ec2-app-role | ⚠️ Review Needed |
| 12 | SSH key management | EC2 | ✅ Key pair infrastructure | ✅ Rotate keys, restrict SSH access, prefer SSM Session Manager | ⚠️ TODO |
| 13 | Instance metadata service (IMDSv2) | EC2 | ✅ IMDSv2 availability | ✅ Enforce IMDSv2, disable IMDSv1 | ❌ Not Configured |

---

### 3. Network Security

| # | Control | Service | AWS Responsibility | Customer Responsibility | Status |
|---|---|---|---|---|---|
| 14 | VPC network isolation | VPC | ✅ Hypervisor-enforced isolation between VPCs | ✅ Design subnets, routing, segmentation | ✅ Configured |
| 15 | Security group rules | EC2, RDS, ALB | ✅ Stateful firewall infrastructure | ✅ Define correct inbound/outbound rules, avoid 0.0.0.0/0 | ⚠️ Review Needed |
| 16 | Network ACLs | VPC Subnets | ✅ NACL infrastructure | ✅ Configure stateless rules as defense-in-depth | ⚠️ Not Hardened |
| 17 | DDoS protection (Layer 3/4) | ALB, CloudFront | ✅ AWS Shield Standard — automatic, no action needed | ✅ Optionally enable Shield Advanced for L7 protection | ❌ Advanced Not Enabled |
| 18 | VPC Flow Logs | VPC | ✅ Flow log infrastructure | ✅ Enable and store flow logs for forensics | ❌ Not Enabled |
| 19 | TLS / HTTPS enforcement | ALB | ✅ TLS termination infrastructure, ACM integration | ✅ Provision certificate, redirect HTTP→HTTPS, set TLS policy | ✅ Configured |

---

### 4. Database Security (RDS)

| # | Control | Service | AWS Responsibility | Customer Responsibility | Status |
|---|---|---|---|---|---|
| 20 | Database host OS patching | RDS | ✅ AWS patches underlying OS automatically | ❌ None | ✅ AWS Managed |
| 21 | Database engine patching | RDS | ✅ Makes patches available; applies during maintenance window | ✅ Schedule maintenance window, test upgrades | ✅ Configured |
| 22 | Encryption at rest | RDS | ✅ KMS integration infrastructure | ✅ Enable encryption at instance creation; manage KMS key | ⚠️ TODO |
| 23 | Encryption in transit (TLS) | RDS | ✅ TLS capability built-in | ✅ Enforce SSL connections in application code and pg_hba | ⚠️ TODO |
| 24 | Network access control | RDS | ✅ Infrastructure | ✅ Place in private subnet, configure sg-db to allow only sg-app | ✅ Private Subnet |
| 25 | Automated backups & PITR | RDS | ✅ Backup infrastructure | ✅ Set retention period, test restore, verify backup integrity | ⚠️ Restore Not Tested |
| 26 | Database user & privilege management | RDS | ❌ None | ✅ Create least-privilege DB users, rotate passwords, avoid using master user in app | ⚠️ TODO |

---

### 5. Data Security (S3)

| # | Control | Service | AWS Responsibility | Customer Responsibility | Status |
|---|---|---|---|---|---|
| 27 | Storage infrastructure durability (11 9s) | S3 | ✅ Automatic data replication across AZs | ❌ None | ✅ AWS Managed |
| 28 | Encryption at rest (SSE) | S3 | ✅ SSE-S3 / SSE-KMS infrastructure | ✅ Enable default bucket encryption; choose key type | ⚠️ TODO |
| 29 | Block Public Access setting | S3 | ✅ Feature availability | ✅ Enable "Block All Public Access" on all buckets | ⚠️ TODO (my-app-assets) |
| 30 | Bucket policies & ACLs | S3 | ✅ Policy evaluation engine | ✅ Author least-privilege bucket policies; disable legacy ACLs | ⚠️ Review Needed |
| 31 | S3 Versioning | S3 | ✅ Versioning infrastructure | ✅ Enable versioning for accidental deletion recovery | ❌ Not Enabled |
| 32 | S3 Access Logging | S3 | ✅ Logging infrastructure | ✅ Enable server access logging and store in separate bucket | ❌ Not Enabled |

---

### 6. Identity & Access Management

| # | Control | Service | AWS Responsibility | Customer Responsibility | Status |
|---|---|---|---|---|---|
| 33 | IAM engine & policy evaluation | IAM | ✅ Policy engine, deny-by-default enforcement | ❌ None | ✅ AWS Managed |
| 34 | IAM least-privilege policies | IAM | ❌ None | ✅ Review and tighten all IAM policies; remove wildcards | ⚠️ Review Needed |
| 35 | MFA enforcement for IAM users | IAM | ✅ MFA capability (TOTP, hardware, passkey) | ✅ Enforce MFA via IAM policy condition; require for console users | ⚠️ TODO |
| 36 | IAM password policy | IAM | ✅ Password policy infrastructure | ✅ Configure minimum length 14+, complexity, rotation | ⚠️ TODO |
| 37 | Root account protection | IAM | ✅ Root account isolation features | ✅ Enable MFA on root, delete/never use root access keys | ✅ MFA Enabled |

---

### 7. Monitoring, Logging & Incident Response

| # | Control | Service | AWS Responsibility | Customer Responsibility | Status |
|---|---|---|---|---|---|
| 38 | CloudTrail API logging | CloudTrail | ✅ CloudTrail infrastructure | ✅ Enable multi-region trail, store in S3 with log integrity | ❌ Not Enabled |
| 39 | CloudWatch alerting | CloudWatch | ✅ Metrics collection infrastructure | ✅ Define alarms for CPU, error rates, unusual API calls | ❌ Not Configured |
| 40 | GuardDuty threat detection | GuardDuty | ✅ ML threat intelligence, finding engine | ✅ Enable GuardDuty in all regions; respond to findings | ❌ Not Enabled |
| 41 | AWS Config compliance rules | AWS Config | ✅ Config recording infrastructure | ✅ Enable Config, define rules (e.g., s3-bucket-public-read-prohibited) | ❌ Not Enabled |
