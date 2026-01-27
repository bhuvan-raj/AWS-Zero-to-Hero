

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
