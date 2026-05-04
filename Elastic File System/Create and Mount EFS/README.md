# 1. AWS EFS Lab (Hands-On)

## Objective

Create EFS and mount it on two EC2 instances.

---

## Architecture

```text
EC2-1 ----\
           ---> EFS ---> Shared files
EC2-2 ----/
```

---

# Step 1: Create Security Groups

## EC2 Security Group

Allow:

* SSH 22 from your IP

## EFS Security Group

Allow inbound:

* TCP 2049 from EC2 Security Group

---

# Step 2: Launch Two EC2 Instances

Launch two Linux instances in same VPC.

Recommended:

* Amazon Linux 2023 / Amazon Linux 2

---

# Step 3: Create EFS

Go to AWS Console:

Amazon Elastic File System

* Create file system
* Select same VPC
* Keep default mount targets
* Attach EFS security group

---

# Step 4: Connect to EC2

```bash
ssh -i key.pem ec2-user@PUBLIC-IP
```

---

# Step 5: Install NFS Utils

## Amazon Linux

```bash
sudo dnf install -y amazon-efs-utils
```

If unavailable:

```bash
sudo dnf install -y nfs-utils
```

---

# Step 6: Create Mount Directory

```bash
sudo mkdir /efs
```

---

# Step 7: Mount EFS

Use your EFS DNS name:

```bash
sudo mount -t efs fs-12345678:/ /efs
```

OR NFS:

```bash
sudo mount -t nfs4 -o nfsvers=4.1 fs-12345678.efs.ap-south-1.amazonaws.com:/ /efs
```

---

# Step 8: Test Shared Storage

On EC2-1:

```bash
echo "Hello from Server1" | sudo tee /efs/test.txt
```

On EC2-2:

```bash
cat /efs/test.txt
```

You should see same file.

---

# Step 9: Auto Mount on Reboot

Edit fstab:

```bash
sudo vi /etc/fstab
```

Add:

```text
fs-12345678:/ /efs efs defaults,_netdev 0 0
```

Then test:

```bash
sudo mount -a
```

---

#  Useful Commands

## Check mount

```bash
df -h
mount | grep efs
```

## Permissions

```bash
ls -ld /efs
chmod
chown
```

---

#  Troubleshooting

## Mount timeout

Check security group port 2049.

## DNS issue

Ensure VPC DNS enabled.

## Permission denied

Check Linux ownership and IAM auth if enabled.

## Wrong subnet

Need mount target in reachable subnet/AZ.

---

#  DevOps Interview Questions

## What is EFS?

Managed scalable NFS shared storage for AWS.

## Can multiple EC2 use same EFS?

Yes.

## Which port does EFS use?

TCP 2049.

## Difference between EFS and EBS?

EFS shared file storage, EBS block volume.

---

# Real-Time DevOps Use Case

Deploy two web servers behind ALB. Mount same EFS on both servers at:

```text
/var/www/html/uploads
```

User uploads become visible on both servers instantly.

---

# Cost Optimization Tips

* Enable lifecycle to IA
* Delete unused data
* Use One Zone only for non-critical workloads

---
