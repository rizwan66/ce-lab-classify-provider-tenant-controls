# Bonus: Capital One Data Breach Analysis (2019)

## Incident Overview

In July 2019, Capital One disclosed that an attacker had accessed approximately **106 million customer records** stored in AWS S3 buckets. The exposed data included names, addresses, phone numbers, credit scores, account numbers, and Social Security numbers. Capital One was fined $80 million by the OCC and settled a class-action lawsuit for $190 million.

This is one of the most instructive cloud breaches in history precisely because it was **not an AWS failure** — it was a customer misconfiguration that the Shared Responsibility Model places squarely on Capital One.

---

## What Was Misconfigured?

### Root Cause: Misconfigured WAF → SSRF → IMDSv1 → IAM Role Credential Theft

The attack chain had three steps:

**Step 1 — WAF Misconfiguration**
Capital One ran a Web Application Firewall (WAF) on an EC2 instance. The WAF had a misconfigured firewall rule that allowed **Server-Side Request Forgery (SSRF)** — a vulnerability where an attacker can make the server issue HTTP requests to internal addresses on the attacker's behalf.

**Step 2 — EC2 Instance Metadata Exploitation (IMDSv1)**
The attacker sent an SSRF request that caused the WAF server to query the **EC2 Instance Metadata Service (IMDS)**:
```
http://169.254.169.254/latest/meta-data/iam/security-credentials/
```
Because IMDSv1 was in use (the default at the time), this endpoint required **no authentication**. Any process — including an SSRF-exploiting attacker — could retrieve temporary IAM credentials belonging to the EC2 instance's attached IAM role.

**Step 3 — IAM Role Credential Abuse → S3 Data Exfiltration**
The EC2 instance's IAM role had **overly broad S3 permissions**. Using the stolen temporary credentials, the attacker called `s3:ListBuckets` and `s3:GetObject` to enumerate and download over 100 million records from Capital One's S3 buckets.

---

## Whose Responsibility Was It?

**Every failure in this chain was Capital One's (customer) responsibility:**

| Failure | AWS or Customer? | Why |
|---|---|---|
| WAF misconfiguration allowing SSRF | **Customer** | Capital One configured the WAF rules |
| IMDSv1 enabled (no token required) | **Customer** | AWS provided IMDSv2 as an option; Capital One chose not to enforce it |
| IAM role with over-broad S3 permissions | **Customer** | IAM policy authoring is entirely customer-controlled |
| No anomaly detection on S3 data access | **Customer** | GuardDuty and CloudTrail were either disabled or not monitored |

AWS's infrastructure — the hypervisor, IMDS service, IAM engine, and S3 storage — all functioned exactly as designed. The breach exploited **customer-managed controls** at every step.

---

## How Could It Have Been Prevented?

### Prevention 1: Enforce IMDSv2 (Token-Required Mode)
IMDSv2 requires a PUT request with a session-oriented token before credentials can be retrieved. SSRF attacks using simple GET requests cannot obtain the token — the attack would have failed at Step 2.

```bash
# Enforce IMDSv2 on all EC2 instances
aws ec2 modify-instance-metadata-options \
  --instance-id <instance-id> \
  --http-tokens required \
  --http-endpoint enabled
```

**This single control would have stopped the Capital One breach.**

### Prevention 2: Apply Least-Privilege IAM Policies
The IAM role attached to the WAF EC2 instance should have had **no S3 permissions** unless the WAF process itself needed to read/write S3. Even if the credentials were stolen, the attacker would have been unable to access S3 buckets.

Bad policy (what Capital One had — simplified):
```json
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*"
}
```

Better policy (least-privilege):
```json
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::specific-bucket-name/specific-prefix/*"
}
```

### Prevention 3: Enable GuardDuty
AWS GuardDuty has a specific finding type — `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS` — that triggers when EC2 instance credentials are used from an IP address outside AWS (i.e., from the attacker's server). This finding would have alerted Capital One during active exfiltration.

### Prevention 4: CloudTrail + S3 Data Event Logging
Enabling CloudTrail data events for S3 would have logged every `GetObject` call. Combined with a CloudWatch alarm on high-volume S3 reads from an unusual source, the breach could have been detected and stopped mid-exfiltration.

### Prevention 5: WAF Rules Preventing SSRF
The root cause was an SSRF-vulnerable WAF configuration. AWS WAF (the managed service) now includes AWS-managed rules that block common SSRF patterns. Using managed rule groups rather than custom rules reduces the chance of misconfiguration.

---

## Connection to This Lab

This breach is a direct real-world example of the gaps identified in this assessment:

| Capital One Failure | This Lab's Gap |
|---|---|
| IMDSv2 not enforced | **GAP-07**: IMDSv2 not enforced on our EC2 instances |
| Over-broad IAM role permissions | **GAP-11** / Matrix #34: IAM policies not reviewed for least-privilege |
| GuardDuty not enabled / monitored | **GAP-04**: GuardDuty not enabled in our account |
| No S3 access anomaly detection | **GAP-03** + **GAP-15**: CloudTrail and CloudWatch alarms missing |

The shared responsibility model does not protect you from misconfiguration — it only defines who is responsible when something goes wrong. In Capital One's case, the responsibility was entirely theirs, as it would be in our architecture with the same gaps.

---

## Key Takeaway

> "The cloud provider secures the infrastructure. You secure everything you put on it."

The Capital One breach was not a failure of AWS. It was a failure to understand and implement customer-side controls — the exact controls this lab is designed to identify. IMDSv2 enforcement, IAM least-privilege, and GuardDuty are not optional hardening steps; they are baseline requirements for any production EC2 workload.
