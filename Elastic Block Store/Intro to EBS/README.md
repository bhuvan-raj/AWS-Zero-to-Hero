# Amazon Elastic Block Storage

## 🧠 What is Block Storage?

Block storage divides data into fixed-size chunks called **blocks**, each with its own address.

👉 It behaves like a **physical hard disk** attached to a server.

---

## 💽 Amazon EBS Overview

Amazon EBS (Elastic Block Store) provides **persistent block storage** for EC2.

### Key Points:
- Acts like a disk for EC2
- Data persists even after instance stop
- Can be detached and reattached

---

## 📊 EBS Volume Types

| Type | Category | Use Case |
|------|--------|----------|
| gp3 | SSD | General workloads |
| gp2 | SSD | Legacy general purpose |
| io1/io2 | SSD | High-performance DB |
| st1 | HDD | Throughput workloads |
| sc1 | HDD | Cold storage |

---

## ⚠️ Important Conditions & Limitations (VERY IMPORTANT FOR INTERVIEWS)

### 1. 📍 Same Availability Zone Requirement
- EBS volume **must be in the same AZ** as the EC2 instance
- ❌ Cannot directly attach across AZs
- ✅ Workaround: Snapshot → Create volume in another AZ

---

### 2. 🔗 Single Instance Attachment (Default)
- By default, **one EBS volume → one EC2 instance**
- ❌ Cannot attach to multiple instances

👉 Exception:
- **Multi-Attach** supported only for:
  - `io1` and `io2` volumes
  - Same AZ only

---

### 3. 🛑 Region Boundaries
- EBS volumes are **region-specific**
- ❌ Cannot directly move across regions

👉 Workaround:
- Snapshot → Copy snapshot → Create volume in new region

---

### 4. 🔄 Detach Before Reattach
- Must **detach volume** before attaching to another instance
- ❌ Cannot attach same volume simultaneously (except Multi-Attach)

---

### 5. ⚙️ Root Volume Behavior
- Root volume is **deleted by default on termination**
- Can disable using:
  - "Delete on termination" option

---

### 6. 📦 Size Constraints
- Minimum: 1 GB
- Maximum: 64 TB

---

### 7. 🖥️ OS-Level Mount Required
- After attaching:
  - You must **format and mount** the volume inside OS
- AWS does NOT auto-mount

---

### 8. 🔒 Encryption Condition
- Once encrypted → cannot be decrypted
- To change:
  - Snapshot → Copy with new encryption

---

### 9. 📸 Snapshot Dependency
- Snapshots stored in S3
- Incremental (only changes stored)

---

### 10. 🚫 Instance Store vs EBS
- Instance store:
  - Temporary
  - Lost on stop/terminate
- EBS:
  - Persistent

---

## ⚙️ EBS Features

- Elastic resizing
- Snapshot backups
- Encryption with KMS
- High availability within AZ
- Multi-Attach (limited)

---

## 🚀 Performance Concepts

| Metric | Meaning |
|------|--------|
| IOPS | Number of operations/sec |
| Throughput | MB/sec |
| Latency | Delay per operation |

---

## 📸 Snapshots

- Stored in S3
- Incremental backups
- Used for:
  - Backup
  - Disaster recovery
  - AMI creation

---

## 🔐 Encryption

- Uses AWS KMS
- Encrypts:
  - Volumes
  - Snapshots
  - Data in transit

---

## 💰 Pricing Factors

- Volume size (GB)
- IOPS (if provisioned)
- Snapshot storage
- Volume type

---

# 🧪 HANDS-ON LAB (AWS CONSOLE)

## 🎯 Objective:
Create, attach, mount, and use an EBS volume.

---

## 🪜 Step 1: Launch EC2 Instance

1. Go to AWS Console
2. Navigate to **EC2 Dashboard**
3. Click **Launch Instance**
4. Choose:
   - AMI: Amazon Linux
   - Instance type: t2.micro
5. Create/select key pair
6. Launch instance

---

## 🪜 Step 2: Create EBS Volume

1. Go to **EC2 → Elastic Block Store → Volumes**
2. Click **Create Volume**
3. Configure:
   - Volume type: gp3
   - Size: 10 GB
   - Availability Zone: SAME as EC2
4. Click **Create Volume**

---

## 🪜 Step 3: Attach Volume

1. Select created volume
2. Click **Actions → Attach Volume**
3. Select your EC2 instance
4. Device name: `/dev/xvdf`
5. Click **Attach**

---

## 🪜 Step 4: Connect to EC2

```bash
ssh -i key.pem ec2-user@<public-ip>
````

---

## 🪜 Step 5: Check Volume

```bash
lsblk
```

👉 You will see new volume like `/dev/xvdf`

---

## 🪜 Step 6: Format Volume

```bash
sudo mkfs -t ext4 /dev/xvdf
```

---

## 🪜 Step 7: Mount Volume

```bash
sudo mkdir /data
sudo mount /dev/xvdf /data
```

---

## 🪜 Step 8: Verify

```bash
df -h
```

---

## 🪜 Step 9: Test Persistence

```bash
cd /data
echo "Hello EBS" > test.txt
```

---

## 🪜 Step 10: Detach & Reattach (Test Condition)

1. Go to Console → Volumes
2. Detach volume
3. Attach to another EC2
4. Mount again → File should exist ✅

---

## 🪜 Step 11: Create Snapshot

1. Select volume
2. Click **Actions → Create Snapshot**
3. Provide name
4. Create

---

## 🪜 Step 12: Create Volume from Snapshot

1. Go to Snapshots
2. Select snapshot
3. Click **Create Volume**
4. Attach to instance

---

# 🧾 Best Practices

* Use **gp3** for most workloads
* Enable **encryption**
* Automate **snapshots**
* Use **io2** for critical DBs
* Clean unused volumes (avoid cost)

---

# 📚 Summary

* EBS = Persistent block storage
* Must be in **same AZ**
* Default = **single instance attachment**
* Snapshots = backup mechanism
* Requires **manual mount inside OS**

---

## 🔁 Quick Interview Revision

* Same AZ? → YES required
* Multi-attach? → Only io1/io2
* Persistent? → YES
* Cross-region? → Snapshot required
* Auto mount? → NO

---
