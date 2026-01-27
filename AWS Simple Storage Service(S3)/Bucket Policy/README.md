# Amazon S3 Bucket Policy

## What is a Bucket Policy?

A **bucket policy** is a **resource-based IAM policy** attached **directly to an S3 bucket**.
It defines **who can access the bucket and its objects, what actions they can perform, and under what conditions**.

Bucket policies are written in **JSON format** and follow the **IAM policy language**.

---

## Key Characteristics of Bucket Policies

* Applied at **bucket level**
* Can control access to:

  * The **bucket**
  * **All or specific objects** in the bucket
* Supports **fine-grained permissions**
* Can grant access to:

  * IAM users
  * IAM roles
  * AWS accounts
  * AWS services
  * Anonymous users (public access)
* Supports **conditions** (IP, VPC endpoint, HTTPS, MFA, etc.)

---

## Structure of a Bucket Policy

A bucket policy consists of:

### 1. Version

Specifies the policy language version.

```json
"Version": "2012-10-17"
```

### 2. Statement

One or more permission rules.

### 3. Effect

* `Allow`
* `Deny`

### 4. Principal

Who is allowed or denied access.

### 5. Action

What S3 actions are allowed or denied.

### 6. Resource

Which bucket or objects the policy applies to.

### 7. Condition (Optional)

Extra restrictions.

---

## Example: Allow an IAM User to Upload and List Objects

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowUserAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/demo-user"
      },
      "Action": [
        "s3:ListBucket",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

---

## Common Use Cases of Bucket Policies

* Grant cross-account access
* Make a bucket public (static website hosting)
* Restrict access to a VPC endpoint
* Enforce HTTPS (`aws:SecureTransport`)
* Limit access by IP address
* Grant AWS services (CloudFront, ELB, logs) access to S3

---

# Access Control List (ACL)

## What is an ACL?

An **ACL (Access Control List)** is a **legacy access control mechanism** in S3 that grants **basic permissions** at:

* Bucket level
* Object level

ACLs assign permissions to predefined groups or AWS accounts.

---

## Permissions Supported by ACL

ACL supports **only limited permissions**:

### Bucket ACL permissions

* READ
* WRITE
* READ_ACP
* WRITE_ACP

### Object ACL permissions

* READ
* READ_ACP
* WRITE_ACP

⚠️ No support for conditions or advanced rules.

---

## Predefined ACL Groups

* Owner
* AuthenticatedUsers
* AllUsers (public)
* LogDelivery group

---

## Example: Make an Object Public Using ACL

```bash
aws s3api put-object-acl \
  --bucket my-bucket \
  --key image.png \
  --acl public-read
```

---

## Important Note on ACLs (AWS Recommendation)

* AWS considers ACLs **legacy**
* **Object Ownership → Bucket owner enforced** disables ACLs
* Best practice: **Use bucket policies and IAM policies instead**

---

# Difference Between Bucket Policy and ACL

| Feature                     | Bucket Policy             | ACL                                 |
| --------------------------- | ------------------------- | ----------------------------------- |
| Type                        | Resource-based IAM policy | Legacy access control               |
| Scope                       | Bucket and objects        | Bucket or individual object         |
| Policy Language             | JSON (IAM policy syntax)  | Simple grant list                   |
| Fine-grained control        | ✅ Yes                     | ❌ No                                |
| Conditions (IP, HTTPS, VPC) | ✅ Supported               | ❌ Not supported                     |
| Cross-account access        | ✅ Yes                     | Limited                             |
| Public access control       | Strong and explicit       | Weak and risky                      |
| AWS Recommendation          | **Preferred & modern**    | **Legacy**                          |
| Disable support             | Cannot be disabled        | Disabled with bucket-owner-enforced |

---

# Bucket Policy vs IAM Policy (Quick Clarification)

* **Bucket policy** → Attached to S3 bucket (resource)
* **IAM policy** → Attached to users, roles, groups

Both are evaluated together.

---

## Security Evaluation Order (Simplified)

1. Explicit **DENY** (bucket policy / IAM)
2. Explicit **ALLOW**
3. Default **DENY**

---

## Exam / Interview One-Liner

> A bucket policy is a resource-based IAM policy attached to an S3 bucket that provides fine-grained, conditional access control, while ACLs are a legacy mechanism offering limited permissions without conditions.

---

## Best Practices (Very Important)

* ✅ Use **bucket policies** for access control
* ❌ Avoid ACLs unless explicitly required
* ✅ Enable **Block Public Access**
* ✅ Use **Object Ownership: Bucket owner enforced**
* ✅ Combine bucket policies with IAM roles


# Lab: Creating a Public Access Bucket using Bucket Policy

## Objective

Enable **public read access** to an S3 bucket using the **bucket policy** via the AWS Management Console.

---

## Prerequisites

* AWS account with **S3 full access**
* Basic knowledge of AWS Console navigation

---

## Step 1: Log in to AWS Console

1. Open [https://aws.amazon.com/console/](https://aws.amazon.com/console/)
2. Sign in using your **AWS credentials**

---

## Step 2: Create a New S3 Bucket

1. Go to **Services → S3**
2. Click **Create bucket**
3. Enter:

   * **Bucket name:** `my-public-bucket-bubu`
   * **Region:** Select your preferred region
4. **Block Public Access settings**:

   * Uncheck **Block all public access**
   * Confirm by checking the warning checkbox
5. Click **Create bucket**

> ✅ Now you have a bucket that **can allow public access**.

---

## Step 3: Upload a Test Object

1. Click on your bucket name: `my-public-bucket-bubu`
2. Click **Upload → Add files**
3. Choose a file, e.g., `test.txt`
4. Click **Upload** (keep default permissions for now)

---

## Step 4: Edit Bucket Policy

1. Go to the bucket **Permissions** tab
2. Scroll down to **Bucket policy**
3. Click **Edit**
4. Paste the following JSON policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-public-bucket-bubu/*"
    }
  ]
}
```

5. Click **Save changes**

> This policy allows **anyone on the internet** to read objects in this bucket.

---

## Step 5: Test Public Access

1. Click on your object (e.g., `test.txt`)
2. Copy the **Object URL**
3. Open a new browser tab and paste the URL
4. You should see your file displayed

> Alternative: Use **curl**:

```bash
curl https://my-public-bucket-bubu.s3.amazonaws.com/test.txt
```

---

## Step 6: Important Security Notes

* Public bucket = **anyone can read files**
* Only use for **static content** (images, website assets)
* Sensitive data must **never** be public
* After lab/testing, you can **re-enable Block Public Access** or delete the bucket

---

## Step 7: Cleanup (Optional)

1. Delete the object: Select → Delete
2. Delete the bucket: **Bucket actions → Delete bucket**

---

## Summary

* **Console GUI Steps**:

  1. Create bucket → Uncheck Block Public Access
  2. Upload object
  3. Add bucket policy JSON → Save
  4. Test URL
* **Bucket policy** allows fine-grained, JSON-based public access control
* Always be cautious with public access


# 🧪 LAB: Grant IAM User Access to an S3 Bucket Using Bucket Policy

---

## 🎯 Lab Objective

To **grant an IAM user access to an S3 bucket using a bucket policy**, verify the access, and understand how resource-based policies work in AWS.

---

## 🧱 Lab Architecture

* **IAM User** → Access granted
* **S3 Bucket** → Resource
* **Bucket Policy** → Controls access

---

## 📋 Prerequisites

* AWS account
* Root or Admin access
* AWS Console access
* Basic understanding of IAM and S3

---

## 🧩 Step 1: Create an IAM User

### 1. Open IAM Console

* AWS Console → **IAM** → **Users**
* Click **Create user**

### 2. User Details

* **User name:** `s3-lab-user`
* Click **Next**

### 3. Permissions

* Select **Attach policies directly**
* ❗ **Do NOT attach any policy**
* Click **Next**

### 4. Review & Create

* Click **Create user**

✅ IAM user created with **no permissions**

---

## 🧩 Step 2: Create an S3 Bucket

### 1. Open S3 Console

* AWS Console → **S3**
* Click **Create bucket**

### 2. Bucket Configuration

* **Bucket name:** `bubu-s3-policy-lab-123`
* **Region:** Same as your account region

### 3. Block Public Access

* Leave **Block all public access = ON**
* (IAM user access is NOT public access)

### 4. Create Bucket

* Click **Create bucket**

✅ Private S3 bucket created

---

## 🧩 Step 3: Upload a Test Object

1. Open the bucket
2. Click **Upload**
3. Upload a file (example: `test.txt`)
4. Click **Upload**

---

## 🧩 Step 4: Get IAM User ARN

1. Go to **IAM → Users**
2. Click `s3-lab-user`
3. Copy **User ARN**

Example:

```
arn:aws:iam::123456789012:user/s3-lab-user
```

---

## 🧩 Step 5: Write the Bucket Policy

### Policy Requirements

We will allow:

* List bucket
* Read objects
* Upload objects

---

### Bucket Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowUserListBucket",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/s3-lab-user"
      },
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::bubu-s3-policy-lab-123"
    },
    {
      "Sid": "AllowUserObjectAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/s3-lab-user"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::bubu-s3-policy-lab-123/*"
    }
  ]
}
```

⚠️ Replace:

* Account ID
* Bucket name
* User name

---

## 🧩 Step 6: Attach the Bucket Policy

1. Go to **S3 → Bucket**
2. Open **Permissions** tab
3. Scroll to **Bucket policy**
4. Paste the policy JSON
5. Click **Save changes**

✅ Bucket policy attached successfully

---

## 🧩 Step 7: Configure IAM User Access

### 1. Create Access Keys

* IAM → Users → `s3-lab-user`
* **Security credentials**
* Create **Access key**
* Choose **CLI**
* Save **Access Key ID & Secret**

---

## 🧩 Step 8: Test Access Using AWS CLI

### Configure CLI

```bash
aws configure
```

Enter:

* Access Key
* Secret Key
* Region
* Output format

---

### Test Commands

#### List bucket

```bash
aws s3 ls s3://bubu-s3-policy-lab-123
```

#### Download object

```bash
aws s3 cp s3://bubu-s3-policy-lab-123/test.txt .
```

#### Upload object

```bash
aws s3 cp newfile.txt s3://bubu-s3-policy-lab-123/
```

✅ All commands should succeed

---

## 🧩 Step 9: Negative Testing (Optional but Important)

### Remove `PutObject` from policy

* Try uploading again
* ❌ Access Denied

This proves **policy enforcement works correctly**

---

## 🔐 Key Concepts Learned

| Concept         | Explanation             |
| --------------- | ----------------------- |
| Bucket Policy   | Resource-based policy   |
| Principal       | IAM user allowed access |
| `ListBucket`    | Bucket-level permission |
| `GetObject`     | Object-level permission |
| `/*`            | Mandatory for objects   |
| Least Privilege | Best practice           |

---
