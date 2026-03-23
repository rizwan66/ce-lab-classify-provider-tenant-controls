# Security Gap Analysis

## Summary

| Priority | Count |
|---|---|
| Critical | 4 |
| High | 5 |
| Medium | 4 |
| Low | 3 |
| **Total Gaps** | **16** |

---

## Critical Gaps — Immediate Action Required (Within 48 Hours)

### GAP-01: S3 Buckets Have No Encryption At Rest
- **Affected Resources:** my-app-assets, my-app-logs, my-app-backups
- **Risk:** If bucket permissions are misconfigured (or credentials are compromised), stored data — including user assets and backup files — is exposed in plaintext.
- **Responsibility:** Customer
- **Control Reference:** Matrix #28
- **Action:** Enable default bucket encryption (SSE-S3 or SSE-KMS) on all three buckets.
  ```bash
  aws s3api put-bucket-encryption \
    --bucket my-app-assets \
    --server-side-encryption-configuration '{
      "Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]
    }'
  ```
- **Effort:** 1 hour

---

### GAP-02: S3 Public Access Not Blocked (my-app-assets)
- **Affected Resources:** my-app-assets bucket
- **Risk:** Any misconfigured object ACL or bucket policy can accidentally make files publicly readable. This is one of the most common causes of S3 data leaks.
- **Responsibility:** Customer
- **Control Reference:** Matrix #29
- **Action:** Enable "Block All Public Access" unless a specific object requires public access (use CloudFront with OAC instead).
  ```bash
  aws s3api put-public-access-block \
    --bucket my-app-assets \
    --public-access-block-configuration \
      "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
  ```
- **Effort:** 30 minutes

---

### GAP-03: CloudTrail Not Enabled
- **Affected Resources:** Entire AWS account
- **Risk:** Without CloudTrail, there is no record of API calls — no audit trail for who created/deleted/modified resources, no forensic capability in the event of a breach. This also blocks compliance with most frameworks (PCI DSS, SOC 2, HIPAA).
- **Responsibility:** Customer
- **Control Reference:** Matrix #38
- **Action:** Create a multi-region CloudTrail trail logging management events. Store logs in a dedicated S3 bucket with log file validation enabled.
  ```bash
  aws cloudtrail create-trail \
    --name org-management-trail \
    --s3-bucket-name my-app-logs \
    --is-multi-region-trail \
    --enable-log-file-validation
  aws cloudtrail start-logging --name org-management-trail
  ```
- **Effort:** 2 hours (including log bucket policy setup)

---

### GAP-04: GuardDuty Not Enabled
- **Affected Resources:** Entire AWS account
- **Risk:** No automated threat detection. Malicious activity (crypto mining on EC2, credential exfiltration, unusual API calls from known-bad IPs) will go undetected until significant damage is done.
- **Responsibility:** Customer
- **Control Reference:** Matrix #40
- **Action:** Enable GuardDuty in all used regions. Set up SNS notification or EventBridge rule to alert on HIGH/CRITICAL findings.
  ```bash
  aws guardduty create-detector --enable --finding-publishing-frequency FIFTEEN_MINUTES
  ```
- **Effort:** 1 hour

---

## High Priority Gaps — Resolve Within 1 Week

### GAP-05: RDS Encryption At Rest Not Enabled
- **Affected Resources:** main-database (RDS PostgreSQL)
- **Risk:** Unencrypted RDS snapshots could expose sensitive data if shared accidentally. Also fails most compliance audits.
- **Responsibility:** Customer
- **Control Reference:** Matrix #22
- **Note:** RDS encryption cannot be enabled on an existing instance — it must be done at creation time. Mitigation requires creating an encrypted snapshot and restoring to a new instance.
- **Action:**
  1. Take a manual snapshot of main-database.
  2. Copy snapshot with encryption enabled (AES-256 via KMS).
  3. Restore new RDS instance from encrypted snapshot.
  4. Update application connection strings to point to new instance.
  5. Decommission unencrypted instance after validation.
- **Effort:** 1 week (includes testing and cutover)

---

### GAP-06: EC2 Guest OS Patching Not Automated
- **Affected Resources:** web-server-1, web-server-2, app-server-1, app-server-2
- **Risk:** Unpatched EC2 instances are vulnerable to known CVEs. The 2021 Log4Shell and 2017 WannaCry incidents demonstrate how rapidly unpatched systems are exploited.
- **Responsibility:** Customer
- **Control Reference:** Matrix #7
- **Action:**
  1. Ensure SSM Agent is running on all EC2 instances.
  2. Configure AWS Systems Manager Patch Manager with a patch baseline for Amazon Linux 2023.
  3. Create a maintenance window for weekly patching (off-peak hours).
  4. Enable Systems Manager Inventory to track installed patches.
- **Effort:** 3 days

---

### GAP-07: IMDSv2 Not Enforced on EC2 Instances
- **Affected Resources:** All four EC2 instances
- **Risk:** IMDSv1 is vulnerable to SSRF attacks. An attacker exploiting an SSRF vulnerability in the Node.js app could use IMDSv1 to retrieve the IAM role credentials (as demonstrated in the Capital One breach — see bonus analysis).
- **Responsibility:** Customer
- **Control Reference:** Matrix #13
- **Action:** Enforce IMDSv2 (token-required mode) on all instances.
  ```bash
  aws ec2 modify-instance-metadata-options \
    --instance-id i-0a1b2c3d4e5f \
    --http-tokens required \
    --http-endpoint enabled
  # Repeat for all four instance IDs
  ```
- **Effort:** 1 hour

---

### GAP-08: MFA Not Enforced for IAM Users
- **Affected Resources:** IAM users with console access
- **Risk:** Compromised IAM user credentials without MFA give an attacker full console and API access. This is a top attack vector in cloud breaches.
- **Responsibility:** Customer
- **Control Reference:** Matrix #35
- **Action:** Attach an IAM policy that denies all actions except setting up MFA for users who have not yet registered an MFA device. Communicate policy to all users.
- **Effort:** 1 day

---

### GAP-09: VPC Flow Logs Not Enabled
- **Affected Resources:** app-vpc
- **Risk:** No visibility into network traffic patterns. Cannot investigate lateral movement, data exfiltration, or unusual connection patterns during an incident.
- **Responsibility:** Customer
- **Control Reference:** Matrix #18
- **Action:** Enable VPC Flow Logs on app-vpc, publish to CloudWatch Logs or S3 (my-app-logs bucket).
  ```bash
  aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-0x1y2z3w \
    --traffic-type ALL \
    --log-destination-type s3 \
    --log-destination arn:aws:s3:::my-app-logs/vpc-flow-logs/
  ```
- **Effort:** 2 hours

---

## Medium Priority Gaps — Resolve Within 1 Month

### GAP-10: S3 Versioning Not Enabled
- **Affected Resources:** my-app-assets, my-app-backups
- **Risk:** Accidental object deletion or overwrites are unrecoverable without versioning.
- **Action:** Enable versioning and set a lifecycle rule to transition old versions to S3 Glacier after 90 days to control cost.
- **Effort:** 1 hour

### GAP-11: Security Groups Need Formal Review
- **Affected Resources:** sg-alb, sg-web, sg-app, sg-db
- **Risk:** Port 80 open on sg-alb redirects HTTP but still accepts connections — review whether HTTP-to-HTTPS redirect is enforced at the ALB listener level so port 80 can be removed from sg-alb.
- **Action:** Audit all security groups. Use AWS Config rule `restricted-ssh` and `restricted-common-ports`. Remove any 0.0.0.0/0 rules other than ports 443 on sg-alb.
- **Effort:** 2 days

### GAP-12: RDS TLS Not Enforced in Application
- **Affected Resources:** main-database, app-server-1, app-server-2
- **Risk:** Database connections may transmit credentials and query data in plaintext on the internal network.
- **Action:** Require SSL in Node.js pg/pg-pool connection config. Use AWS RDS CA bundle. Set `rds.force_ssl=1` in the parameter group.
- **Effort:** 1 day

### GAP-13: Database Users / Least Privilege Not Reviewed
- **Affected Resources:** main-database
- **Risk:** If the application uses the master database user, a SQL injection could give an attacker full database admin access.
- **Action:** Create a dedicated application DB user with only SELECT/INSERT/UPDATE/DELETE on the required schema. Rotate the master password and store it in AWS Secrets Manager.
- **Effort:** 1 day

---

## Low Priority Gaps — Resolve Within 90 Days

### GAP-14: AWS Config Not Enabled
- **Action:** Enable AWS Config with the AWS Foundational Security Best Practices standard. Provides ongoing compliance posture monitoring.
- **Effort:** 1 day

### GAP-15: CloudWatch Alarms Not Configured
- **Action:** Create alarms for: EC2 CPU > 90%, ALB 5xx error rate > 1%, RDS free storage < 20%, GuardDuty HIGH findings.
- **Effort:** 1 day

### GAP-16: S3 Access Logging Disabled
- **Action:** Enable server access logging on all buckets, delivering logs to my-app-logs.
- **Effort:** 1 hour

---

## Prioritized Remediation Roadmap

| Week | Action |
|---|---|
| Week 1 (Days 1-2) | GAP-01: Encrypt all S3 buckets; GAP-02: Block S3 public access; GAP-03: Enable CloudTrail; GAP-04: Enable GuardDuty; GAP-07: Enforce IMDSv2 |
| Week 1 (Days 3-5) | GAP-06: Deploy SSM Patch Manager; GAP-08: Enforce IAM MFA; GAP-09: Enable VPC Flow Logs |
| Week 2 | GAP-05: Begin RDS encryption migration (snapshot, restore, validate) |
| Week 3-4 | GAP-10 through GAP-13: Medium priority controls |
| Month 2-3 | GAP-14 through GAP-16: Low priority controls + ongoing compliance monitoring |
