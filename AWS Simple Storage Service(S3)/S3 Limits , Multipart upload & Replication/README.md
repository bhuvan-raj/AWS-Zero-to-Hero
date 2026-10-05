

# Object Size & Storage Limits in Amazon S3

## 1. Object Size Limits in S3

Amazon S3 stores data as **objects inside buckets**. While storage is virtually unlimited, **individual objects are constrained by size limits**.

---

### 1.1 Maximum Object Size

* **Maximum size of a single S3 object: 5 TB**

Key points:

* This is a **hard limit** and cannot be increased.
* Applies regardless of upload method.
* Objects larger than 5 TB **cannot be stored in S3**.

---

### 1.2 Single PUT Upload Limit

* Maximum size using a **single PUT operation: 5 GB**

Implications:

* Uploading objects **larger than 5 GB requires multipart upload**.
* Attempting a single PUT above 5 GB results in failure.

---

### 1.3 Number of Buckets

* Default limit: **100 buckets per AWS account**

Additional notes:

* Bucket names are **globally unique** across AWS.
* Limit increases can be requested via AWS Support.
* Best practice is to **use fewer buckets with logical prefixes**.

---

### 1.4 Number of Objects per Bucket

* **Unlimited number of objects** per bucket

S3 automatically scales to handle:

* Millions or billions of objects
* No performance degradation due to object count

---

### 1.5 Total Storage per Bucket

* **Unlimited storage per bucket**

Important:

* No need to provision capacity.
* You only pay for **what you store**.
* Growth is automatic and elastic.

---

# 2: Multipart Uploads in Amazon S3

## 2.1 What is Multipart Upload?

Multipart upload is a feature that allows **large objects to be uploaded as multiple smaller parts**, which are then **assembled by S3 into a single object**.

It is:

* **Required** for objects larger than 5 GB
* **Recommended** for objects larger than 100 MB

---

## 2.2 How Multipart Upload Works (Workflow)

1. **Initiate Multipart Upload**

   * S3 returns a unique **Upload ID**.
   * All parts reference this ID.

2. **Upload Parts**

   * Each part is uploaded independently.
   * Parts can be uploaded **in parallel**.
   * Each part has a **part number**.

3. **Complete Multipart Upload**

   * S3 assembles all uploaded parts.
   * Final object becomes available.

4. **Abort Multipart Upload (Optional)**

   * Cancels upload and removes uploaded parts.
   * Prevents unnecessary storage charges.

---

## 2.3 Multipart Upload Size Limits

| Parameter               | Limit                   |
| ----------------------- | ----------------------- |
| Minimum part size       | 5 MB (except last part) |
| Maximum part size       | 5 GB                    |
| Maximum number of parts | 10,000                  |
| Maximum object size     | 5 TB                    |

---

## 2.4 Benefits of Multipart Upload

### Reliability

* Failed uploads require retrying only the failed part.

### Performance

* Parallel uploads significantly increase throughput.

### Network Efficiency

* Handles unstable or slow connections gracefully.

### Resume Capability

* Uploads can resume using the Upload ID.

### Cost Control

* Failed uploads can be aborted to avoid charges.

---

# 3: Object Replication in Amazon S3

## 3.1 What is Object Replication?

Object replication is an S3 feature that **automatically copies objects** from a **source bucket** to one or more **destination buckets**, either within the same region or across regions.

Replication is:

* **Asynchronous**
* **Object-level**
* Based on **replication rules**

---

## 3.2 Types of S3 Replication

### Same-Region Replication (SRR)

* Source and destination buckets are in the **same AWS region**.
* Used for:

  * Data segregation
  * Security isolation
  * Analytics workloads

### Cross-Region Replication (CRR)

* Buckets are in **different AWS regions**.
* Used for:

  * Disaster recovery
  * Compliance
  * Latency reduction

---

## 3.3 Replication Prerequisites

* Versioning **must be enabled** on both buckets.
* Destination bucket must already exist.
* Proper **IAM role and permissions** required.
* Replication applies only to **new objects** by default.

---

## 3.4 Replication – What Gets Copied

| Item                | Replicated                 |
| ------------------- | -------------------------- |
| New object versions | Yes                        |
| Object metadata     | Yes                        |
| Object tags         | Optional                   |
| ACLs                | Optional                   |
| Delete markers      | Optional                   |
| Existing objects    | No (use Batch Replication) |

---

## 3.5 Replication of Deletes

* Deleting an object creates a **delete marker** in versioned buckets.
* Replication of delete markers is **optional**.
* Permanent deletes are **not replicated**.

---

## 3.6 Batch Replication

Batch Replication is used to:

* Replicate **existing objects**
* Retry **failed replications**
* Apply new rules to old data

Common scenarios:

* Replication enabled late
* Compliance-driven data sync
* Backfilling historical data

---

## 3.7 Common Use Cases

* Disaster recovery across regions
* Regulatory compliance
* Multi-region application data
* Centralized log storage
* Data sovereignty requirements

---

## 3.8 Best Practices

* Always enable versioning before replication.
* Use prefixes or tags to control replication scope.
* Monitor replication metrics.
* Secure IAM roles and KMS permissions.
* Combine replication with lifecycle policies.
* Use Batch Replication for historical data.


# **S3 Replication Lab (Same Account)**

### **Objective**

Automatically replicate objects from a source S3 bucket to a destination S3 bucket **within the same AWS account**.

---

###  Create Buckets**

1. Log in to the **AWS Management Console** → go to **S3**.
2. Click **Create bucket**.

   * **Source bucket**: `s3-replication-source`
   * **Destination bucket**: `s3-replication-destination`
   * Keep **Region** same or different (same account).
3. For **both buckets**, enable **Versioning**:

   * Open the bucket → **Properties** → **Bucket Versioning** → **Enable**
---

###  Configure Replication Rule**

1. Go to **S3 → Source Bucket → Management → Replication Rules → Create replication rule**.
2. **Step 1: Rule scope**

   * Name the rule (e.g., `ReplicateAllObjects`).
   * Choose **Apply to all objects** (or filter by prefix/tag if desired).
3. **Step 2: Destination**

   * Choose **Same AWS account**.
   * Select **Destination bucket**: `s3-replication-destination`.
4. **Step 3: IAM Role**

   * Choose **Create a New role**
5. **Step 4: Additional options**

   * Enable **Replicate delete markers** if you want deletions to replicate.
6. Review → click **Create rule**.

---

### **Step 4: Test Replication**

1. Upload a file to **source bucket**:

2. Check **destination bucket** in the console after a few seconds.

   * The file should appear automatically.

3. Optional: Delete the file in the source bucket and check if delete marker appears in the destination (if enabled).

---

### **Lab Outcome**

* Source bucket: `s3-replication-source`
* Destination bucket: `s3-replication-destination`
* Objects are automatically replicated in near real-time.
* IAM Role handles permissions for replication.

---
# AWS S3 Cross-Account Replication Lab

## Objective

Configure Amazon S3 to automatically replicate objects from a **source bucket in Account A** to a **destination bucket in Account B**.

### Architecture

```text
┌──────────────────────┐             ┌──────────────────────┐
│      Account A       │             │      Account B       │
│       SOURCE         │             │    DESTINATION       │
│                      │             │                      │
│  Source S3 Bucket    │             │ Destination S3 Bucket│
│         │            │             │          ▲            │
│         │            │             │          │            │
│         ▼            │             │          │            │
│ S3 Replication Role  ├─────────────┼──────────┘            │
│                      │ Replication │                       │
└──────────────────────┘             └──────────────────────┘
```

> **Important:** We use a single replication IAM role in **Account A**. Amazon S3 assumes this role to perform replication.

---

# Prerequisites

You need:

* **Account A** → Source account
* **Account B** → Destination account
* One S3 bucket in each account
* Versioning enabled on both buckets
* Permission to create IAM roles and edit S3 bucket policies

For this lab, use example names:

```text
Account A:
Source Bucket → bubu-source-bucket-12345

Account B:
Destination Bucket → bubu-destination-bucket-67890
```

Replace these with your own globally unique bucket names.

---

# Step 1: Create the Source Bucket

Log in to **Account A**.

Go to:

```text
AWS Console
   ↓
S3
   ↓
Create bucket
```

Create:

```text
Bucket name:
bubu-source-bucket-12345
```

Keep the default settings for this lab.

Create the bucket.

---

# Step 2: Create the Destination Bucket

Log in to **Account B**.

Go to:

```text
S3
   ↓
Create bucket
```

Create:

```text
Bucket name:
bubu-destination-bucket-67890
```

Create the bucket.

---

# Step 3: Enable Versioning on Both Buckets

Versioning is required for S3 replication.

### Account A

```text
S3
 ↓
Source Bucket
 ↓
Properties
 ↓
Bucket Versioning
 ↓
Enable
```

### Account B

```text
S3
 ↓
Destination Bucket
 ↓
Properties
 ↓
Bucket Versioning
 ↓
Enable
```

Verify that both show:

```text
Bucket Versioning: Enabled
```

---

# Step 4: Create the Replication IAM Role in Account A

Now log in to **Account A**.

Go to:

```text
IAM
 ↓
Roles
 ↓
Create role
```

For the trusted entity, choose:

```text
Custom trust policy
```

Use:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Create the role with:

```text
Role name:
s3-cross-account-replication-role
```

The important point here is:

```text
S3 Service
    ↓
AssumeRole
    ↓
s3-cross-account-replication-role
```

We are **not** creating a replication role in Account B.

---

# Step 5: Give the Role Permission to Read the Source Bucket

Still in **Account A**, open:

```text
IAM
 ↓
Roles
 ↓
s3-cross-account-replication-role
 ↓
Add permissions
 ↓
Create inline policy
```

Choose **JSON** and add:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetReplicationConfiguration",
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::SOURCE-BUCKET"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObjectVersionForReplication",
        "s3:GetObjectVersionAcl",
        "s3:GetObjectVersionTagging"
      ],
      "Resource": "arn:aws:s3:::SOURCE-BUCKET/*"
    }
  ]
}
```

Replace:

```text
SOURCE-BUCKET
```

with your actual source bucket.

For example:

```text
arn:aws:s3:::bubu-source-bucket-12345
```

and:

```text
arn:aws:s3:::bubu-source-bucket-12345/*
```

Name the policy:

```text
S3ReplicationSourceAccess
```

---

# Step 6: Give the Role Permission to Write to Account B

This is the important cross-account part.

Still in **Account A**, attach another inline policy to:

```text
s3-cross-account-replication-role
```

Use:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete",
        "s3:ReplicateTags"
      ],
      "Resource": "arn:aws:s3:::DESTINATION-BUCKET/*"
    }
  ]
}
```

Replace:

```text
DESTINATION-BUCKET
```

with the Account B bucket.

For example:

```text
arn:aws:s3:::bubu-destination-bucket-67890/*
```

Name the policy:

```text
S3ReplicationDestinationAccess
```

---

# Step 7: Configure the Destination Bucket Policy

Now log in to **Account B**.

Go to:

```text
S3
 ↓
Destination Bucket
 ↓
Permissions
 ↓
Bucket policy
```

Add:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReplicationFromAccountA",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT-A-ID:role/s3-cross-account-replication-role"
      },
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete",
        "s3:ReplicateTags"
      ],
      "Resource": "arn:aws:s3:::DESTINATION-BUCKET/*"
    }
  ]
}
```

Replace:

```text
ACCOUNT-A-ID
```

with the AWS Account ID of Account A.

Replace:

```text
DESTINATION-BUCKET
```

with the Account B bucket name.

For example:

```json
"Principal": {
  "AWS": "arn:aws:iam::111111111111:role/s3-cross-account-replication-role"
}
```

---

# Step 8: Get the Replication Role ARN

Go back to **Account A**:

```text
IAM
 ↓
Roles
 ↓
s3-cross-account-replication-role
```

Copy the ARN.

It will look like:

```text
arn:aws:iam::111111111111:role/s3-cross-account-replication-role
```

You'll use this when creating the replication rule.

---

# Step 9: Create the Replication Rule

Go to **Account A**:

```text
S3
 ↓
Source Bucket
 ↓
Management
 ↓
Replication rules
 ↓
Create replication rule
```

Give it a name:

```text
replicate-to-account-b
```

### Rule scope

For the lab, choose:

```text
Apply to all objects in the bucket
```

### Destination

Choose:

```text
Another AWS account
```

Enter:

```text
Account ID:
ACCOUNT-B-ID
```

Then select/specify the destination bucket:

```text
arn:aws:s3:::bubu-destination-bucket-67890
```

---

# Step 10: Select the IAM Role

For the replication IAM role, select:

```text
Choose from existing IAM role
```

Select:

```text
s3-cross-account-replication-role
```

The role is in **Account A**.

Remember:

```text
Account A
    │
    └── s3-cross-account-replication-role
             │
             │
             ▼
       Account B Bucket
```

---

# Step 11: Replication Options

For the lab, you can enable:

```text
Delete marker replication
```

If you want to demonstrate replication of existing objects, enable:

```text
Replicate existing objects
```

However, for a simple lab, I recommend **testing with a newly uploaded object first**.

Create the replication rule.

---

# Step 12: Test Replication

Go to **Account A**:

```text
S3
 ↓
Source Bucket
 ↓
Upload
```

Create a file:

```text
test.txt
```

with:

```text
Hello from Account A
```

Upload it.

Then go to **Account B**:

```text
S3
 ↓
Destination Bucket
```

You should eventually see:

```text
test.txt
```

The object has been replicated.

---

# Step 13: Test Versioning

Modify the file in Account A.

For example:

```text
Version 1:
Hello from Account A
```

Then upload another version:

```text
Version 2:
Hello from Account A - Updated
```

Check:

```text
Source Bucket
 ↓
Object
 ↓
Versions
```

You should see multiple versions.

Then check the destination bucket and verify that the replicated versions are present.

---

# Final Architecture

```text
                         ACCOUNT A
                    ┌───────────────────┐
                    │                   │
                    │  Source Bucket    │
                    │                   │
                    │   test.txt        │
                    │       │           │
                    └───────┼───────────┘
                            │
                            │ S3 assumes role
                            ▼
                 ┌──────────────────────┐
                 │ s3-cross-account-    │
                 │ replication-role     │
                 └──────────┬───────────┘
                            │
                            │ ReplicateObject
                            │ ReplicateDelete
                            │ ReplicateTags
                            ▼
                    ┌───────────────────┐
                    │    ACCOUNT B      │
                    │                   │
                    │ Destination      │
                    │ Bucket            │
                    │                   │
                    │ test.txt ✓        │
                    └───────────────────┘
```

### The key concept students should remember

```text
S3 in Account A
      │
      │ assumes
      ▼
IAM Role in Account A
      │
      │ reads
      ▼
Source Bucket
      │
      │ replicates
      ▼
Destination Bucket in Account B
```





## **Step 7: Lab Outcome**

* Objects in the source bucket (Account A) are **replicated to a destination bucket in Account B** automatically.
* Delete markers can be replicated if enabled.
* IAM roles ensure **secure cross-account replication**.

---

✅ **Notes / Best Practices**

* Use **unique object prefixes** if replicating multiple types of data.
* Replication can be **same-region** or **cross-region**.
* Replication **does not copy existing objects** by default; you need to enable **replicate existing objects** if required.
* For **large-scale replication**, monitor **replication metrics in S3 → Metrics**.

---
