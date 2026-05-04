# AWS EFS Study Notes (Amazon Elastic File System)

## 1. What is AWS EFS?

Amazon Web Services **Amazon Elastic File System (EFS)** is a **fully managed, scalable Network File System (NFS)** that can be mounted simultaneously on multiple Linux-based Amazon EC2 instances. It provides shared file storage that grows and shrinks automatically as files are added or removed.

---

# 2. Key Features of EFS

## Fully Managed

AWS handles infrastructure, patching, replication within Availability Zones, and durability.

## Elastic Storage

No need to pre-provision capacity. Storage automatically scales.

## Shared File System

Multiple EC2 instances can mount the same EFS at the same time.

## NFS Protocol

Uses standard **NFSv4.1 / NFSv4.0** protocol.

## High Availability

Can be accessed from multiple Availability Zones inside a VPC.

## Secure

Supports:

* Security groups
* IAM policies
* Encryption at rest
* Encryption in transit

---

# 3. Common Use Cases

* Web server shared content
* WordPress uploads shared across multiple servers
* Container persistent storage (EKS/ECS)
* Big data analytics
* DevOps shared config/scripts
* Home directories
* Backup repositories

---

# 4. EFS Performance Modes

## General Purpose

Low latency, ideal for web apps and CMS.

## Max I/O

Higher aggregate throughput, suitable for analytics and large workloads.

---

# 5. Throughput Modes

## Bursting Throughput

Default. Throughput scales with stored data.

## Provisioned Throughput

Manually set throughput independent of storage size.

## Elastic Throughput

Automatically adapts to workload demand.

---

# 6. Storage Classes

## Standard

Frequently accessed files.

## Infrequent Access (IA)

Lower cost for files not often accessed.

## Archive

Lowest cost for rarely accessed data.

Lifecycle policies can move files automatically.

---

# 7. Important Ports

| Service | Port |
| ------- | ---- |
| NFS     | 2049 |

Allow inbound TCP 2049 from client EC2 security group.

---

# 8. EFS vs EBS vs S3

| Feature        | EFS          | EBS                       | S3                         |
| -------------- | ------------ | ------------------------- | -------------------------- |
| Type           | File Storage | Block Storage             | Object Storage             |
| Shared Mount   | Yes          | No (single attach mostly) | API access                 |
| Mount on Linux | Yes          | Yes                       | No direct mount by default |
| Auto Scale     | Yes          | Manual                    | Yes                        |
| Use Case       | Shared files | OS disks/DB               | Backups/static files       |

---

