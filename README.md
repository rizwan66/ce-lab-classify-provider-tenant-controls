# Lab M8.01 - Classify Controls into Provider vs Tenant Responsibility

**Repository:** [https://github.com/cloud-engineering-bootcamp/ce-lab-classify-provider-tenant-controls](https://github.com/cloud-engineering-bootcamp/ce-lab-classify-provider-tenant-controls)

**Activity Type:** Individual  
**Estimated Time:** 45-60 minutes

## Learning Objectives

- [ ] Review AWS Shared Responsibility Model components
- [ ] Classify security controls by responsibility (AWS vs customer)
- [ ] Identify responsibility for different service models (IaaS, PaaS, SaaS)
- [ ] Document security controls and responsibilities for a three-tier architecture

## Prerequisites

- [ ] Completed Module 8 Lesson 1
- [ ] AWS account with existing resources (VPC, EC2, RDS, S3)
- [ ] Basic understanding of IAM, security groups, and encryption

## Introduction

The Shared Responsibility Model is foundational to cloud security. Misunderstanding where your responsibilities begin and end leads to security gaps and compliance failures. In this lab, you'll classify security controls for a real architecture and document who is responsible for each control.

## Scenario

You've been hired as a cloud security engineer for a startup. They have a three-tier web application running on AWS:
- **Web tier:** Application Load Balancer, EC2 instances
- **App tier:** EC2 instances running Node.js
- **Data tier:** RDS PostgreSQL database

Your task is to create a responsibility matrix showing which security controls are managed by AWS and which are your responsibility.

## Your Task

**What you'll create:**
- Responsibility matrix mapping controls to AWS or customer
- Classification of services by model (IaaS, PaaS, SaaS)
- Documentation of security responsibilities for each tier
- Gap analysis identifying controls that need implementation

**Success criteria:**
- [ ] Responsibility matrix completed with at least 20 controls
- [ ] All services correctly classified by service model
- [ ] Identified at least 5 customer-responsible controls that need implementation
- [ ] Documented findings in markdown file
- [ ] Submitted to student portal

**Time limit:** 45-60 minutes

## Step-by-Step Instructions

### Step 1: Inventory Your AWS Resources

1. Log into AWS Console
2. Navigate to each service and list your resources:
   - EC2 instances (web and app tier)
   - RDS database
   - S3 buckets
   - Security groups
   - IAM roles

3. Document in `security-inventory.md`:
   ```markdown
   # AWS Resource Inventory

   ## Compute
   - EC2 Instance: i-abc123 (web-server-1)
   - EC2 Instance: i-def456 (app-server-1)

   ## Database
   - RDS PostgreSQL: main-database

   ## Storage
   - S3 Bucket: my-app-assets

   ## Networking
   - VPC: vpc-xyz789
   - Security Groups: sg-web, sg-app, sg-db

   ## Identity
   - IAM Role: ec2-app-role
   ```

### Step 2: Create Responsibility Matrix Template

Create a file called `responsibility-matrix.md`:

```markdown
# Security Responsibility Matrix

| Control | Service | AWS Responsibility | Customer Responsibility | Status |
|---------|---------|-------------------|-------------------------|--------|
| Physical data center security | All | ✅ Physical security, guards, access control | ❌ None | ✅ AWS Managed |
| Hypervisor security | EC2 | ✅ Hypervisor patching, isolation | ❌ None | ✅ AWS Managed |
| Guest OS patching | EC2 | ❌ None | ✅ Install security updates on EC2 | ⚠️ TODO |
| RDS OS patching | RDS | ✅ OS and database patching | ❌ None | ✅ AWS Managed |
| S3 bucket encryption | S3 | ✅ Encryption infrastructure | ✅ Enable encryption, manage keys | ⚠️ TODO |
| ... | ... | ... | ... | ... |
```

### Step 3: Classify Each Service by Model

**Identify whether each service is IaaS, PaaS, or SaaS:**

| Service | Model | Justification |
|---------|-------|---------------|
| EC2 | IaaS | Customer manages OS, applications, data |
| RDS | PaaS | AWS manages OS, customer manages data and configuration |
| S3 | PaaS/SaaS | AWS manages infrastructure, customer manages data and access |
| Lambda | FaaS/PaaS | AWS manages OS and runtime, customer manages code |
| ALB | PaaS | AWS manages infrastructure, customer configures routing |

**Add this to your `responsibility-matrix.md` file.**

### Step 4: Map Security Controls to Responsibilities

For each control category, identify AWS and customer responsibilities:

#### Network Security

| Control | AWS | Customer | Status |
|---------|-----|----------|--------|
| VPC isolation | ✅ Hypervisor isolation | ✅ VPC configuration, subnets | ✅ Configured |
| Security groups | ✅ Infrastructure | ✅ Rules configuration | ⚠️ Review needed |
| NACLs | ✅ Infrastructure | ✅ Rules configuration | ✅ Configured |
| DDoS protection | ✅ AWS Shield Standard | ✅ Enable Shield Advanced (optional) | ❌ Not enabled |

#### Compute Security (EC2)

| Control | AWS | Customer | Status |
|---------|-----|----------|--------|
| Host OS | ✅ Patching and security | ❌ None | ✅ AWS Managed |
| Guest OS | ❌ None | ✅ Patching, hardening, antivirus | ⚠️ TODO |
| Application security | ❌ None | ✅ Secure coding, dependencies | ⚠️ TODO |
| IAM role | ✅ Infrastructure | ✅ Policy configuration | ✅ Configured |
| Instance isolation | ✅ Hypervisor | ❌ None | ✅ AWS Managed |

#### Database Security (RDS)

| Control | AWS | Customer | Status |
|---------|-----|----------|--------|
| Database OS | ✅ OS patching | ❌ None | ✅ AWS Managed |
| Database engine | ✅ Patching (with maintenance window) | ✅ Schedule maintenance, test | ✅ Configured |
| Encryption at rest | ✅ KMS infrastructure | ✅ Enable encryption, manage keys | ⚠️ TODO |
| Network access | ✅ Infrastructure | ✅ Security groups, subnet placement | ✅ Private subnet |
| Backup | ✅ Automated backups | ✅ Configure retention, test restore | ⚠️ Test restore |

#### Data Security (S3)

| Control | AWS | Customer | Status |
|---------|-----|----------|--------|
| Storage infrastructure | ✅ Redundancy, durability | ❌ None | ✅ AWS Managed |
| Encryption at rest | ✅ KMS infrastructure | ✅ Enable encryption | ⚠️ TODO |
| Bucket policies | ✅ Infrastructure | ✅ Configure policies | ⚠️ Review needed |
| Public access | ✅ Infrastructure | ✅ Block Public Access setting | ⚠️ TODO |
| Versioning | ✅ Infrastructure | ✅ Enable versioning | ❌ Not enabled |

### Step 5: Identify Gaps and Prioritize

Review your matrix and identify controls with status ⚠️ TODO or ❌ Not enabled.

**Create a gap analysis:**

```markdown
# Security Gap Analysis

## Critical Gaps (Immediate Action Required)
1. **S3 buckets not encrypted**
   - Risk: Data breach if bucket misconfigured
   - Priority: Critical
   - Action: Enable default encryption on all buckets

2. **EC2 guest OS not patched regularly**
   - Risk: Exploitation of known vulnerabilities
   - Priority: Critical
   - Action: Implement AWS Systems Manager Patch Manager

## High Priority Gaps
3. **Security groups need review**
   - Risk: Overly permissive rules
   - Priority: High
   - Action: Review all security groups, remove 0.0.0.0/0 rules

4. **RDS encryption not enabled**
   - Risk: Database exposure if snapshots are shared
   - Priority: High
   - Action: Create encrypted snapshot, restore to new encrypted instance

## Medium Priority Gaps
5. **S3 versioning not enabled**
   - Risk: Accidental deletion without recovery
   - Priority: Medium
   - Action: Enable versioning on all buckets
```

### Step 6: Document Findings

Create a comprehensive report:

```markdown
# AWS Shared Responsibility Assessment Report

## Executive Summary
This report documents the security responsibilities for our three-tier application running on AWS. We identified 5 critical/high priority gaps that require immediate attention.

## Architecture Overview
- Web tier: ALB + EC2 (public subnet)
- App tier: EC2 (private subnet)
- Data tier: RDS PostgreSQL (private subnet)

## Service Model Classification
- EC2: IaaS (high customer responsibility)
- RDS: PaaS (medium customer responsibility)
- S3: PaaS (medium customer responsibility)
- ALB: PaaS (low customer responsibility)

## Responsibility Matrix
[Insert completed table from Step 4]

## Gap Analysis
[Insert gap analysis from Step 5]

## Recommendations
1. Enable S3 default encryption (1 day)
2. Deploy AWS Systems Manager for patch management (3 days)
3. Conduct security group audit (2 days)
4. Migrate RDS to encrypted instance (1 week)
5. Enable S3 versioning on all buckets (1 day)

## Conclusion
Clear understanding of responsibilities is critical. We must implement the 5 identified controls within the next 2 weeks to achieve minimum security baseline.
```

### Step 7: Verify With AWS Documentation

Cross-reference your matrix with AWS documentation:

1. Open [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
2. Verify your classifications match AWS guidance
3. Update any discrepancies in your matrix

### Step 8: Create Visual Diagram (Bonus)

Use draw.io or similar tool to create a visual representation:

```
┌─────────────────────────────────────────────┐
│         AWS RESPONSIBILITY                  │
│  Physical Security, Hardware, Hypervisor    │
└─────────────────────────────────────────────┘
                    ↑
                    │
┌─────────────────────────────────────────────┐
│       CUSTOMER RESPONSIBILITY               │
│  Guest OS, Applications, Data, IAM,         │
│  Encryption, Network Config, Firewall       │
└─────────────────────────────────────────────┘
```

## Submission

Submit the following to the student portal:

1. **`responsibility-matrix.md`** - Completed matrix with 20+ controls
2. **`security-inventory.md`** - AWS resource inventory
3. **`gap-analysis.md`** - Identified gaps and priorities
4. **`assessment-report.md`** - Comprehensive report

## Verification Checklist

Before submitting, verify:

- [ ] At least 20 security controls documented
- [ ] All services classified by model (IaaS/PaaS/SaaS/FaaS)
- [ ] AWS vs customer responsibilities clearly identified
- [ ] Gaps identified and prioritized
- [ ] Recommendations include timeline estimates
- [ ] Cross-referenced with AWS official documentation

## Bonus Challenges (Optional)

### Bonus 1: Real-World Breach Analysis

Research the Capital One breach (2019) and answer:
- What was misconfigured?
- Whose responsibility was it (AWS or Capital One)?
- How could it have been prevented?

Add your analysis to `bonus-breach-analysis.md`.

### Bonus 2: Automate Compliance Checking

Write a script to check if S3 buckets are encrypted:

```bash
#!/bin/bash
# check-s3-encryption.sh

echo "Checking S3 bucket encryption status..."

aws s3api list-buckets --query 'Buckets[*].Name' --output text | \
while read bucket; do
  encryption=$(aws s3api get-bucket-encryption --bucket "$bucket" 2>&1)
  
  if [[ $encryption == *"ServerSideEncryptionConfigurationNotFoundError"* ]]; then
    echo "❌ $bucket - NOT ENCRYPTED"
  else
    echo "✅ $bucket - ENCRYPTED"
  fi
done
```

### Bonus 3: Create Compliance Dashboard

Build a simple dashboard showing:
- Total AWS services in use
- % with encryption enabled
- % with MFA enabled
- Security group rules count

## Troubleshooting

### Issue: Can't determine who is responsible for a control

**Solution:** Ask these questions:
1. Can I configure this setting in AWS Console? → Likely customer responsibility
2. Does AWS documentation mention "customer managed"? → Customer responsibility
3. Is this infrastructure-level? → Likely AWS responsibility

### Issue: Service doesn't fit neatly into IaaS/PaaS/SaaS

**Solution:** Some services are hybrid. For example:
- Lambda is FaaS (function-as-a-service), closest to PaaS
- ECS on EC2 is IaaS (you manage OS), but ECS on Fargate is PaaS

### Issue: Don't know if control is implemented

**Solution:** Check AWS Console:
- S3 encryption: Bucket → Properties → Default encryption
- RDS encryption: Database → Configuration → Encryption
- MFA: IAM → Users → Security credentials

## Learning Reflection

After completing this lab, consider:

1. **What surprised you about the Shared Responsibility Model?**
2. **Which customer responsibilities are most commonly misunderstood?**
3. **How would you explain shared responsibility to a non-technical stakeholder?**
4. **What's the biggest risk if customers don't understand their responsibilities?**

## Additional Resources

- [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
- [AWS Security Best Practices](https://aws.amazon.com/architecture/security-identity-compliance/)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)
- [Capital One Breach Post-Mortem](https://krebsonsecurity.com/2019/07/capital-one-breach-spotlights-need-for-defense-in-depth/)

**Good luck! 🔒**
