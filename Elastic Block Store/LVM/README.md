
# 📘 Disk Management: fdisk vs LVM + Hands-on Lab

---

## 🔹 What is `fdisk`?

* A tool used to create **disk partitions**
* Works at **disk level**
* Creates fixed-size partitions like:

  ```
  /dev/xvdf1, /dev/xvdf2
  ```
* Simple and traditional

---

## 🔹 What is LVM (Logical Volume Manager)?

* Advanced storage management layer
* Works **on top of partitions or disks**
* Provides **flexible, resizable storage**

### 🔧 Components:

* **PV (Physical Volume)** → Disk or partition
* **VG (Volume Group)** → Pool of storage
* **LV (Logical Volume)** → Usable partition

---

## ⚖️ Key Differences

| Feature          | fdisk              | LVM                                |
| ---------------- | ------------------ | ---------------------------------- |
| Type             | Partitioning tool  | Storage management system          |
| Flexibility      | Fixed size         | Resizable                          |
| Resize support   | Difficult          | Easy                               |
| Snapshot support | ❌ No               | ✅ Yes                              |
| Disk combination | ❌ No               | ✅ Yes (multiple disks into one VG) |
| Use case         | Basic partitioning | Production, scalable storage       |

---

## 🎯 Interview One-liner

> fdisk is used for static disk partitioning, whereas LVM provides flexible storage with resizing, snapshots, and disk aggregation.

---

# 🧪 LVM Lab (Hands-on)

## 🧱 Scenario

You attached a new EBS volume (e.g., `/dev/xvdf`) and want to configure LVM.

---

## 🔹 Step 1: Check disk

```bash
lsblk
```

---

## 🔹 Step 2: Create Physical Volume (PV)

```bash
sudo pvcreate /dev/xvdf
sudo pvdisplay
```

---

## 🔹 Step 3: Create Volume Group (VG)

```bash
sudo vgcreate my_vg /dev/xvdf
sudo vgdisplay
```

---

## 🔹 Step 4: Create Logical Volume (LV)

```bash
sudo lvcreate -n my_lv -L 5G my_vg
sudo lvdisplay
```

---

## 🔹 Step 5: Create filesystem

```bash
sudo mkfs.ext4 /dev/my_vg/my_lv
```

---

## 🔹 Step 6: Create mount point

```bash
sudo mkdir /data
```

---

## 🔹 Step 7: Mount volume

```bash
sudo mount /dev/my_vg/my_lv /data
df -h
```

---

## 🔹 Step 8: Permanent mount

```bash
sudo blkid
sudo vi /etc/fstab
```

Add:

```
UUID=<uuid>  /data  ext4  defaults  0  2
```

Apply:

```bash
sudo mount -a
```

---

# 📈 Extend Logical Volume (lvextend)

## 🔹 Step 1: Check free space

```bash
sudo vgs
```

---

## 🔹 Step 2: Extend LV

```bash
sudo lvextend -L +2G /dev/my_vg/my_lv
```

---

## 🔹 Step 3: Resize filesystem

```bash
sudo resize2fs /dev/my_vg/my_lv
```

---

## ✅ Shortcut (recommended)

```bash
sudo lvextend -r -L +2G /dev/my_vg/my_lv
```

---

# 📉 Reduce Logical Volume (lvreduce)

⚠️ **Risky operation — follow order strictly**

---

## 🔹 Step 1: Unmount

```bash
sudo umount /data
```

---

## 🔹 Step 2: Check filesystem

```bash
sudo e2fsck -f /dev/my_vg/my_lv
```

---

## 🔹 Step 3: Reduce filesystem FIRST

```bash
sudo resize2fs /dev/my_vg/my_lv 6G
```

---

## 🔹 Step 4: Reduce LV

```bash
sudo lvreduce -L 6G /dev/my_vg/my_lv
```

---

## 🔹 Step 5: Mount back

```bash
sudo mount /dev/my_vg/my_lv /data
df -h
```

---

## ✅ Shortcut (safer)

```bash
sudo lvreduce -r -L 6G /dev/my_vg/my_lv
```

---

# ⚠️ Important Rules

* Extend → **LV first → FS next**
* Reduce → **FS first → LV next**
* Always backup before reducing

---

# 🚀 Real-world Use Cases

* Combine multiple EBS volumes into one VG
* Increase disk without downtime
* Used in production systems for scalability

---

# 🧠 Quick Memory

* **fdisk = fixed**
* **LVM = flexible**

