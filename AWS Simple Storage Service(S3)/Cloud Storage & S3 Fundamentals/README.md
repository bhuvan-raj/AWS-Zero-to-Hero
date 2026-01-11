# Module 1: Cloud Storage & Amazon S3 Fundamentals

## 1. What is Cloud Storage?

Cloud Storage is a model of data storage where digital data is stored on **remote servers managed by a cloud provider** and accessed over the internet.

### Key Characteristics
- On-demand scalability
- High durability and availability
- No physical hardware management
- Pay-as-you-go pricing
- Global accessibility

---

## 2. Types of Cloud Storage

### 2.1 Object Storage
- Data stored as objects
- Each object contains:
  - Data
  - Metadata
  - Unique identifier (object key)
- Ideal for unstructured data

**Example:** Amazon S3

---

### 2.2 Block Storage
- Data split into blocks
- Attached to compute instances
- Requires filesystem formatting

**Example:** Amazon EBS

---

### 2.3 File Storage
- Data organized as files and folders
- Supports shared access
- Uses NFS/SMB protocols

**Example:** Amazon EFS

---

## 3. Object vs Block vs File Storage

| Feature | Object (S3) | Block (EBS) | File (EFS) |
|------|-----------|-----------|-----------|
| Structure | Objects | Blocks | Files |
| Scalability | Very high | Limited | High |
| Cost | Low | Higher | Moderate |
| Use Case | Backups, static content | Databases | Shared apps |

---

## 4. Introduction to Amazon S3

Amazon Simple Storage Service (Amazon S3) is a **highly scalable object storage service** designed for durability, security, and performance.

### Key Features
- 11 nines (99.999999999%) durability
- Unlimited storage
- Integrated security controls
- Native AWS service integration

---

## 5. S3 Global Service vs Regional Buckets

- Amazon S3 is a **global service**
- Buckets are created in a **specific AWS region**
- Bucket names must be **globally unique**
- Data physically resides in the selected region

---

## 6. S3 Pricing Overview (High Level)

Pricing is based on:
- Storage used per GB
- Number of requests (PUT, GET, LIST)
- Data transfer out
- Storage class used

---

## 7. What is an S3 Bucket?

An S3 bucket is a **logical container** used to store objects.

### Bucket Characteristics
- Globally unique name
- Region-specific
- Unlimited objects
- Flat namespace (no real folders)

---

## 8. Hands-on Lab 1: Create an S3 Bucket Using AWS Console

### Steps
1. Login to AWS Management Console
2. Navigate to **Amazon S3**
3. Click **Create bucket**
4. Enter:
   - Bucket name (globally unique)
   - AWS region
5. Leave defaults for:
   - Block public access
   - Object ownership
6. Click **Create bucket**

---

## 9. Upload and Download Objects Using AWS Console

### Upload
1. Open the bucket
2. Click **Upload**
3. Add files
4. Click **Upload**

### Download
1. Select the object
2. Click **Download**

---

## 10. Hands-on Lab 2: Create an S3 Bucket Using AWS CLI

### Prerequisites
- AWS account
- IAM user with S3 permissions
- AWS CLI installed
- Access key and secret key

---

### Step 1: Install AWS CLI
Verify installation:
```bash
aws --version
````

---

### Step 2: Configure AWS CLI

```bash
aws configure
```

Enter:

* AWS Access Key ID
* AWS Secret Access Key
* Default region name
* Default output format (json)

---

### Step 3: Create an S3 Bucket Using CLI

#### For regions other than us-east-1:

```bash
aws s3api create-bucket \
  --bucket my-unique-bucket-name-123 \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1
```

#### For us-east-1:

```bash
aws s3api create-bucket --bucket my-unique-bucket-name-123
```

---

### Step 4: Verify Bucket Creation

```bash
aws s3 ls
```

---

## 11. Upload an Object to S3 Using AWS CLI

### Upload a File

```bash
aws s3 cp myfile.txt s3://my-unique-bucket-name-123/
```

### Verify Upload

```bash
aws s3 ls s3://my-unique-bucket-name-123/
```

---

## 12. Download an Object Using AWS CLI

```bash
aws s3 cp s3://my-unique-bucket-name-123/myfile.txt .
```

---

## 13. Key Learnings from This Module

* Difference between cloud storage types
* Why Amazon S3 is object storage
* Global vs regional behavior of S3
* Bucket fundamentals
* Performing S3 operations using:

  * AWS Console
  * AWS CLI

---

## 14. Interview-Oriented Notes

* S3 is object-based, not file-based
* Buckets are region-specific but globally named
* No traditional filesystem in S3
* CLI uses APIs under the hood

---

## Next Module

**Module 2: S3 Architecture & Core Concepts**

* Objects, keys, prefixes
* Metadata
* Durability and availability

---
