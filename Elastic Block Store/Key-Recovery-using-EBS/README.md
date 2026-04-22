

# 🔐 LAB: Key Recovery in AWS Using EBS

## 🎯 Objective

Recover access to an EC2 instance when:

* SSH key pair is lost
* `.pem` file missing
* Permission issues block login

👉 Technique: **Detach root EBS → attach to another instance → modify `authorized_keys`**

---

# 🧰 Prerequisites

* A running EC2 instance (locked out)
* Another working EC2 instance (helper instance)
* Access to AWS Console (**Amazon EC2**)

---

# 🔹 Step 1: Identify Root Volume

1. Go to EC2 → Instances
2. Select the locked instance
3. Go to **Storage tab**
4. Note the **Root EBS Volume ID**

---

# 🔹 Step 2: Stop the Instance

```text
Instance → Actions → Stop
```

👉 Must stop before detaching root volume

---

# 🔹 Step 3: Detach Root EBS Volume

1. Go to Volumes
2. Select root volume
3. Click **Detach Volume**

---

# 🔹 Step 4: Attach Volume to Helper Instance

1. Select the same volume
2. Click **Attach Volume**
3. Choose helper instance
4. Device name:

```text
/dev/xvdf
```

---

# 🔹 Step 5: Connect to Helper Instance

```bash
ssh ec2-user@helper-instance-ip
```

---

# 🔹 Step 6: Identify Attached Volume

```bash
lsblk
```

Example:

```text
xvda   → root disk (helper instance)
xvdf   → attached volume (target instance)
```

---

# 🔹 Step 7: Mount the Volume

## Create mount directory

```bash
sudo mkdir /mnt/recovery
```

## Mount

```bash
sudo mount /dev/xvdf1 /mnt/recovery
```

👉 Sometimes partition may be `/dev/xvdf` instead of `xvdf1`

---

# 🔹 Step 8: Modify authorized_keys

Navigate:

```bash
cd /mnt/recovery/home/ec2-user/.ssh/
```

Edit:

```bash
sudo vi authorized_keys
```

👉 Add your **new public key**

Example:

```text
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC...
```

---

# 🔹 Step 9: Fix Permissions (IMPORTANT)

```bash
sudo chmod 700 /mnt/recovery/home/ec2-user/.ssh
sudo chmod 600 /mnt/recovery/home/ec2-user/.ssh/authorized_keys
sudo chown -R 1000:1000 /mnt/recovery/home/ec2-user/.ssh
```

---

# 🔹 Step 10: Unmount Volume

```bash
sudo umount /mnt/recovery
```

---

# 🔹 Step 11: Detach from Helper Instance

* Go to AWS Console → Volumes → Detach

---

# 🔹 Step 12: Reattach to Original Instance

* Attach as:

```text
/dev/xvda
```

👉 Must be same root device name

---

# 🔹 Step 13: Start Original Instance

```text
Instance → Start
```

---

# 🔹 Step 14: SSH into Instance

```bash
ssh -i new-key.pem ec2-user@your-instance-ip
```

✅ Access restored

---




