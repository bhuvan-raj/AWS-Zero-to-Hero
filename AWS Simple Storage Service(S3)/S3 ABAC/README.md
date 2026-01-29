# Bucket ABAC in Amazon S3 

---

## 1. What is ABAC?

**ABAC (Attribute-Based Access Control)** is an authorization model where **access decisions are made using attributes (tags)** instead of hard-coding identities or resource names.

In AWS, ABAC mainly uses:

* **IAM principal tags** (user / role tags)
* **Resource tags** (S3 bucket or object tags)
* **Policy conditions**

Access is allowed **only when attributes match**.

---

## 2. Traditional Access Control Problem (RBAC Limitation)

### Without ABAC (Traditional IAM)

You write policies like:

```
Allow user A → bucket A
Allow user B → bucket B
Allow user C → bucket C
```

Problems:

* Thousands of users
* Thousands of buckets
* Policy explosion
* Hard to maintain
* Manual updates required

This model does **not scale**.

---

## 3. What Bucket ABAC Solves

With ABAC:

> “Users can access only those S3 buckets whose tags match their IAM tags.”

No bucket names.
No user names.
No policy rewrite.

Only **tags**.

---

## 4. Core Concept of Bucket ABAC

Access is decided based on:

```
IAM principal tag == S3 bucket tag
```

Example:

| Entity    | Tag                |
| --------- | ------------------ |
| IAM User  | Department=Finance |
| S3 Bucket | Department=Finance |

✅ Access allowed

If mismatch:

❌ Access denied

---

## 5. Attributes Used in S3 ABAC

### 1️⃣ Principal Attributes

* IAM user tags
* IAM role tags
* Session tags

Example:

```
Department = HR
Environment = Prod
```

---

### 2️⃣ Resource Attributes

* S3 bucket tags
* Object tags

Example:

```
Department = HR
Environment = Prod
```

---

### 3️⃣ Conditions in Policy

Used to compare attributes dynamically.

---

## 6. ABAC vs RBAC (Very Important)

| Feature             | RBAC  | ABAC       |
| ------------------- | ----- | ---------- |
| Access based on     | Roles | Attributes |
| Scalability         | Poor  | Excellent  |
| Policy size         | Large | Small      |
| Dynamic access      | ❌     | ✅          |
| Enterprise friendly | ❌     | ✅          |

AWS strongly recommends ABAC for large environments.

---

## 7. Bucket ABAC Architecture

```
IAM User / Role
   ↓ (has tags)
S3 Bucket
   ↓ (has tags)
IAM Policy with condition
   ↓
Access decision
```

---

## 8. Example Scenario (Real Enterprise Use Case)

Company structure:

* Finance team
* HR team
* Engineering team

Each team has:

* Its own S3 buckets
* Its own IAM users

Instead of writing separate policies:

Use ABAC.

---

## 9. Tagging Strategy Example

### IAM Users / Roles

```
Department = Finance
```

### S3 Buckets

```
Department = Finance
```

---

## 10. Sample ABAC IAM Policy for S3 Bucket

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Department": "${aws:PrincipalTag/Department}"
        }
      }
    }
  ]
}
```

### What this means:

* Resource = any bucket
* Access allowed only if:

  ```
  bucket tag == user tag
  ```

---

## 11. Important Condition Keys Used in ABAC

| Condition Key            | Meaning              |
| ------------------------ | -------------------- |
| aws:PrincipalTag/tag-key | IAM user/role tag    |
| aws:ResourceTag/tag-key  | Bucket or object tag |
| aws:RequestTag/tag-key   | Tag in request       |
| aws:TagKeys              | Allowed tag keys     |

---

## 12. Bucket-Level vs Object-Level ABAC

### Bucket-Level ABAC

* Uses bucket tags
* Controls ListBucket, GetBucketLocation

### Object-Level ABAC

* Uses object tags
* Controls GetObject, PutObject

Both can be combined.

---

## 13. Object Tag–Based ABAC Example

Object tag:

```
Project = Alpha
```

IAM role tag:

```
Project = Alpha
```

Policy condition:

```json
"StringEquals": {
  "s3:ExistingObjectTag/Project":
  "${aws:PrincipalTag/Project}"
}
```

This allows access only to objects with matching tags.

---

## 14. Why ABAC is Powerful in S3

* No bucket-specific policies
* No identity-specific policies
* One policy works for thousands of buckets
* Access automatically updates when tags change

Change tag → access changes automatically.

---

## 15. Real Production Use Cases

* Multi-team S3 environments
* Multi-account AWS organizations
* SaaS platforms
* Data lakes
* Shared service accounts
* Centralized logging buckets

---

## 16. Security Benefits

* Least privilege by design
* Reduced human error
* Strong separation of teams
* Easier audits
* Tag-based governance

---

## 17. Common Mistakes

* Forgetting to tag buckets
* Inconsistent tag keys
* Case-sensitive tag mismatch
* Allowing wildcard without conditions
* Mixing RBAC and ABAC incorrectly

---

## 18. ABAC with AWS Organizations

ABAC works extremely well with:

* AWS Organizations
* SCPs
* Mandatory tagging rules
* Automated provisioning

Enterprise-grade design.

---

## 19. Interview One-Line Answer

> **Bucket ABAC in S3 is an authorization model where access to buckets and objects is controlled dynamically using IAM principal tags and S3 resource tags instead of explicit bucket names.**

---

## 20. Final Summary

* ABAC = Attribute-based authorization
* Uses tags, not identities
* Highly scalable
* Recommended for large AWS environments
* Best practice for modern IAM design
* 




# 🧪 LAB: Implement Bucket ABAC (Attribute-Based Access Control)

---

## 🎯 Objective

To allow access to an S3 bucket **only when IAM principal tags match S3 bucket tags**, without specifying user or role ARNs in the bucket policy.

---

## 🧠 Concept Used

* Attribute-Based Access Control (ABAC)
* IAM Principal Tags
* S3 Resource Tags
* Bucket Policy Conditions

---

## 🏗 Architecture

```
IAM User (tagged)
        │
        ▼
   Bucket Policy
        │
        ▼
S3 Bucket (tagged)
```

Access is allowed **only if tags match**.

---

## 🔧 Pre-requisites

* AWS account
* IAM user with:

  * Programmatic access
  * Console access
* AWS-managed policies only

---

# 🔹 LAB DETAILS

| Component  | Value           |
| ---------- | --------------- |
| IAM User   | abac-user       |
| IAM Tag    | Project = dev   |
| S3 Bucket  | abac-dev-bucket |
| Bucket Tag | Project = dev   |

---

# STEP 1 — Create IAM User

1. Go to **IAM → Users**
2. Click **Create user**
3. User name:

   ```
   abac-user
   ```
4. Enable:

   * AWS Management Console access
5. Click **Next**

---

## Attach policy

Attach AWS managed policy:

```
AmazonS3ReadOnlyAccess
```

(Used only to test; bucket policy will enforce ABAC.)

Click **Next → Create user**

---

# STEP 2 — Add TAG to IAM User

1. Open **IAM → Users → abac-user**
2. Go to **Tags tab**
3. Click **Add tag**

```
Key   : Project
Value : dev
```

Save.

✅ IAM principal now has an attribute.

---

# STEP 3 — Create S3 Bucket

1. Go to **S3**
2. Click **Create bucket**

```
Bucket name: abac-dev-bucket
Region: ap-south-1
```

3. Keep **Block all public access ON**
4. Create bucket

---

# STEP 4 — Add TAG to S3 Bucket

1. Open the bucket
2. Go to **Properties**
3. Scroll to **Tags**
4. Add tag:

```
Key   : Project
Value : dev
```

Save.

---

# STEP 5 — Add Bucket Policy (ABAC Policy)

1. Go to **Permissions tab**
2. Open **Bucket policy**
3. Paste the following policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "BucketABACAccess",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::abac-dev-bucket",
        "arn:aws:s3:::abac-dev-bucket/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:PrincipalTag/Project": "${s3:ResourceTag/Project}"
        }
      }
    }
  ]
}
```

Save policy.

---

# 🔥 What this policy means

AWS evaluates:

```
IAM User Tag      → Project = dev
Bucket Tag        → Project = dev
Condition         → MATCH
```

✅ Access allowed.

No user ARN is mentioned anywhere.

---

# STEP 6 — Test Access (Positive Case)

1. Log in as **abac-user**
2. Open **S3**
3. Click bucket **abac-dev-bucket**

✅ You can:

* List bucket
* Download objects

---

# STEP 7 — Negative Test (Mismatch)

## Change IAM tag

IAM → Users → abac-user → Tags

Change:

```
Project = prod
```

---

## Test again

Try opening the bucket.

❌ Result:

```
Access Denied
```

No policy changes were made.

Only tag changed.

---

# ✅ This confirms ABAC is working

Access is **fully controlled by tags**.

---

# 🧠 Key Observations

| Action           | Result         |
| ---------------- | -------------- |
| Tag match        | Access allowed |
| Tag mismatch     | Access denied  |
| No ARN in policy | Yes            |
| Scalable         | Yes            |
| Dynamic          | Yes            |

---

# 🔐 Important Notes

* Tag keys are **case-sensitive**
* Principal tags work for:

  * IAM users
  * IAM roles
* Bucket policy is mandatory
* Tags alone do nothing without policy logic

---

# 🧠 Interview-ready explanation

> Bucket ABAC uses IAM principal tags and S3 resource tags.
> Access is granted dynamically when tag values match, without hard-coding users or roles in bucket policies.

---

# ✅ Real-world usage

* Multi-team environments
* Dev / Test / Prod isolation
* SaaS tenant separation
* CI/CD role-based bucket access
* Large enterprise AWS accounts

---
