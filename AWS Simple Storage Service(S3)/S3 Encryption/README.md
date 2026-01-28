# Encryption in AWS S3

---

## 1. What is Encryption?

**Encryption** is the process of converting readable data (**plaintext**) into unreadable data (**ciphertext**) using a mathematical algorithm and a **key**, so that unauthorized users cannot understand the data even if they gain access to it.

In simple terms:

> **Encryption = Data protection by scrambling content using a key**

Only someone with the **correct key** can decrypt and read the original data.

---

## 2. Why Encryption is Important in S3

Amazon S3 stores **critical and sensitive data**, such as:

* Application logs
* User uploads
* Backups
* Database dumps
* Personally Identifiable Information (PII)

Without encryption:

* Anyone who gets access to the storage can read the data
* Compliance requirements (HIPAA, PCI-DSS, GDPR, ISO, SOC) are violated

AWS strongly recommends encryption, and **many services enforce it by default today**.

---

## 3. What is an Encryption Key?

An **encryption key** is a **secret value** used by the encryption algorithm to:

* Encrypt data (lock)
* Decrypt data (unlock)

### Key Points:

* The **same data encrypted with different keys produces different ciphertext**
* Protecting the key is **more important than protecting the data**
* If the key is lost → data is **permanently unrecoverable**

---

## 4. Types of Encryption Keys

### 4.1 Symmetric Keys

* Same key is used for encryption and decryption
* Faster and commonly used for large data
* Used by S3 for data encryption

**Example:** AES-256 (Advanced Encryption Standard)

### 4.2 Asymmetric Keys

* Uses a **key pair**

  * Public key → encrypt
  * Private key → decrypt
* Slower, used mainly for key exchange

**Example:** RSA, ECC
*(Not directly used by S3 to encrypt objects)*

---

## 5. Encryption Types in AWS S3

S3 supports **two major encryption categories**:

1. **Encryption at Rest**
2. **Encryption in Transit**

---

## 6. Encryption at Rest in S3 (Most Important)

Encryption at rest protects data **while it is stored on disk** inside AWS data centers.

### AWS S3 supports **Server-Side Encryption (SSE)** and **Client-Side Encryption**

---

## 7. Server-Side Encryption (SSE)

With SSE:

* You upload **plaintext data**
* AWS encrypts the object **after receiving it**
* AWS stores encrypted data
* Decryption happens automatically when you download

### You don’t manage encryption logic — AWS does.

---

## 8. Types of Server-Side Encryption in S3

### 8.1 SSE-S3 (S3-Managed Keys)

**Overview:**

* AWS fully manages encryption keys
* Uses **AES-256**
* No additional cost
* Simplest option

**How it works:**

1. You upload an object
2. S3 generates a unique data key
3. Data is encrypted
4. Key is stored and managed by AWS internally

**Characteristics:**

* No visibility into keys
* No key rotation control
* No audit trail on key usage

**Use case:**

* Basic encryption
* Non-regulated workloads
* When simplicity > control

---

### 8.2 SSE-KMS (AWS KMS-Managed Keys)

**Overview:**

* Uses **AWS Key Management Service (KMS)**
* You control the master key (CMK / KMS Key)
* Supports auditing and fine-grained access control

**How it works:**

1. S3 requests a data key from KMS
2. KMS encrypts the data key using a KMS key
3. S3 encrypts the object using the data key
4. Encrypted data key is stored with the object

**Key Advantages:**

* Key usage logged in **CloudTrail**
* Automatic key rotation (for AWS-managed keys)
* IAM policies control who can use the key
* Can disable or delete keys (data becomes unreadable)

**Limitations:**

* Slight latency due to KMS calls
* KMS API request costs apply

**Use case:**

* Compliance-heavy workloads
* Regulated environments
* When auditing and access control are required

---

### 8.3 SSE-C (Customer-Provided Keys)

**Overview:**

* You provide the encryption key with every request
* AWS does NOT store the key

**How it works:**

1. You send the object + encryption key
2. S3 encrypts data using the key
3. Key is discarded after encryption
4. Same key must be provided to download

**Risks & Limitations:**

* Key loss = permanent data loss
* HTTPS is mandatory
* Not supported in AWS Console (CLI/SDK only)
* Hard to manage at scale

**Use case:**

* Extreme control requirements
* Rarely used in practice

---

## 9. Client-Side Encryption

With client-side encryption:

* Data is encrypted **before it reaches S3**
* AWS never sees plaintext data or keys

**Options:**

* AWS Encryption SDK
* Custom encryption logic

**Pros:**

* Maximum security
* AWS cannot decrypt your data

**Cons:**

* You manage keys
* You manage encryption/decryption logic
* Higher operational complexity

**Use case:**

* Highly sensitive data
* Zero-trust security model

---

## 10. Default Encryption in S3 Buckets

You can configure **default encryption** at the bucket level:

* Automatically encrypts all new objects
* Users don’t need to specify encryption headers

Supported defaults:

* SSE-S3
* SSE-KMS

**Important:**

* Default encryption does NOT encrypt existing objects
* You must re-upload or copy existing objects

---

## 11. Encryption in Transit (Data in Motion)

S3 encrypts data **while being transferred** using:

* **HTTPS (TLS)**
* Secure endpoint communication

**Key Points:**

* HTTPS is enabled by default
* You can enforce HTTPS using a bucket policy
* Prevents man-in-the-middle attacks

---

## 12. Enforcing Encryption in S3

You can enforce encryption using:

* **Bucket Policies**
* **SCPs (Service Control Policies)**

Example enforcement ideas:

* Deny uploads without encryption headers
* Allow only SSE-KMS
* Block unencrypted object uploads

---

## 13. Relationship Between KMS Keys and S3 Objects

* Each object has its own **data key**
* Data key is encrypted using a **KMS key**
* Disabling the KMS key makes all related objects unreadable
* Deleting the KMS key is irreversible

---

## 14. Common Misconceptions

| Misconception               | Reality                               |
| --------------------------- | ------------------------------------- |
| S3 is encrypted by default  | Only if default encryption is enabled |
| Encryption slows S3 heavily | Negligible performance impact         |
| SSE-S3 is less secure       | It is secure but less controllable    |
| KMS stores your data        | KMS stores only keys, never data      |

---

## 15. Summary Table

| Encryption Type | Key Owner | Audit | Cost | Control   |
| --------------- | --------- | ----- | ---- | --------- |
| SSE-S3          | AWS       | ❌     | Free | Low       |
| SSE-KMS         | You / AWS | ✅     | Paid | High      |
| SSE-C           | You       | ❌     | Free | Very High |
| Client-Side     | You       | ❌     | Free | Maximum   |

---






# S3 Encryption using AWS KMS (SSE-KMS)

---

## 1. Objective

To enable **server-side encryption using AWS KMS (SSE-KMS)** on an Amazon S3 bucket and **verify upload/download access using an IAM user via AWS Console**, using **AWS-managed IAM policies** and minimal custom configuration.

---

## 2. Prerequisites

* AWS account with admin access
* Browser access to AWS Console
* IAM user with **console login enabled**

---

## 3. Services Used

* Amazon S3
* AWS KMS
* AWS IAM

---

## 4. High-Level Flow

1. Create KMS key (GUI)
2. Create S3 bucket
3. Enable default encryption (SSE-KMS)
4. Create IAM user (console access)
5. Attach **AWS-managed policies**
6. Update **KMS key policy** (GUI)
7. Test upload & download via **S3 Console**
8. Perform negative test

---

## 5. Step 1: Create a Customer Managed KMS Key (GUI)

1. Go to **AWS Console → KMS**
2. Click **Create key**
3. **Key type**: Symmetric
4. **Key usage**: Encrypt and decrypt
5. Click **Next**
6. **Alias**:

   ```
   s3-kms-gui-lab
   ```
7. **Key administrators**:

   * Select your **admin user**
8. **Key users**:

   * Leave empty for now
9. Finish key creation

📌 *Do not add IAM user yet — we’ll do it after user creation.*

---

## 6. Step 2: Create an S3 Bucket

1. Go to **S3 → Create bucket**
2. Bucket name:

   ```
   bubu-s3-kms-gui-lab
   ```
3. Region: **Same region as KMS key**
4. **Block all public access** → ON
5. Create bucket

---

## 7. Step 3: Enable Default Encryption (SSE-KMS)

1. Open the bucket
2. Go to **Properties**
3. Scroll to **Default encryption**
4. Enable encryption
5. Encryption type:
   **AWS Key Management Service (AWS KMS)**
6. Select KMS key:

   ```
   s3-kms-gui-lab
   ```
7. Save changes

✅ All future uploads are now encrypted using KMS automatically.

---

## 8. Step 4: Create IAM User (Console Access)

1. Go to **IAM → Users → Create user**
2. Username:

   ```
   s3-kms-console-user
   ```
3. Select **Provide user access to the AWS Management Console**
4. Choose:

   * **IAM-generated password**
   * Require password reset (optional)
5. Click **Next**

---

## 9. Step 5: Attach Existing AWS-Managed Policies

Attach the following **AWS-managed policies**:

### 1️⃣ AmazonS3FullAccess

(Required for bucket and object access)

### 2️⃣ AWSKeyManagementServicePowerUser

(Required for using KMS keys)

✔ No custom IAM policy needed

Click **Next → Create user**

---

## 10. Step 6: Update KMS Key Policy (GUI — Critical)

> Even with managed policies, **KMS will deny access unless the key policy allows the user**.

### Add IAM User as Key User

1. Go to **KMS → Customer managed keys**
2. Open:

   ```
   s3-kms-gui-lab
   ```
3. Go to **Key users** tab
4. Click **Add**
5. Select:

   ```
   s3-kms-console-user
   ```
6. Save changes

✅ Now the IAM user can **encrypt and decrypt objects** using this key.

---

## 11. Step 7: Login as IAM User

1. Sign out from admin account
2. Login using:

   * IAM user console URL
   * Username: `s3-kms-console-user`
   * Password

---

## 12. Step 8: Upload Object via S3 Console (Encryption Test)

1. Go to **S3 → bubu-s3-kms-gui-lab**
2. Click **Upload**
3. Add any file
4. Click **Upload**

### Verify Encryption

1. Click the uploaded object
2. Go to **Properties**
3. Check **Server-side encryption**

You should see:

```
Encryption type: AWS-KMS
KMS key: s3-kms-gui-lab
```

✅ Encryption is working.

---

## 13. Step 9: Download Object (Decryption Test)

1. Select the object
2. Click **Download**

✅ File downloads successfully
Decryption happens transparently.

---

## 14. Step 10: Negative Test (Important Concept Proof)

### Remove KMS Access

1. Login as **admin**
2. Go to **KMS → s3-kms-gui-lab**
3. Remove:

   ```
   s3-kms-console-user
   ```

   from **Key users**
4. Save changes

---

### Test Again as IAM User

1. Login as IAM user
2. Try to download the object

❌ **Access Denied**

📌 This proves:

* S3 permissions exist
* **KMS controls decryption**
* Key policy overrides IAM permissions

---

## 15. Key Takeaways (Exam + Real World)

* SSE-KMS is **server-side encryption**
* Default encryption applies automatically
* **AWS-managed policies are NOT enough alone**
* KMS key policy is mandatory
* Console users can fully test encryption behavior

---
