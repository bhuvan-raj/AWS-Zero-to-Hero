

# 📘 **Difference between fdisk and LVM**

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
* Components:

  * Physical Volume (PV)
  * Volume Group (VG)
  * Logical Volume (LV)

---

## ⚖️ **Key Differences**

| Feature          | fdisk              | LVM                                |
| ---------------- | ------------------ | ---------------------------------- |
| Type             | Partitioning tool  | Storage management system          |
| Flexibility      | Fixed size         | Resizable                          |
| Resize support   | Difficult          | Easy                               |
| Snapshot support | ❌ No               | ✅ Yes                              |
| Disk combination | ❌ No               | ✅ Yes (multiple disks into one VG) |
| Use case         | Basic partitioning | Production, scalable storage       |

---

## 🎯 **Interview One-liner**

> “fdisk is used for static disk partitioning, whereas LVM provides flexible storage management with resizing, snapshots, and disk aggregation.”

---

# 🧪 **LVM Lab Steps (Hands-on)**

## 🧱 Scenario

You attached a new EBS volume (e.g., `/dev/xvdf`) and want to configure LVM.

---

## 🔹 Step 1: Check disk

```bash id="fwsj1m"
lsblk
```

---

## 🔹 Step 2: Create Physical Volume (PV)

```bash id="f3kt0p"
sudo pvcreate /dev/xvdf
```

Verify:

```bash id="6dxn0g"
sudo pvdisplay
```

---

## 🔹 Step 3: Create Volume Group (VG)

```bash id="i6z0b5"
sudo vgcreate my_vg /dev/xvdf
```

Verify:

```bash id="ej47v4"
sudo vgdisplay
```

---

## 🔹 Step 4: Create Logical Volume (LV)

```bash id="szlbqr"
sudo lvcreate -n my_lv -L 5G my_vg
```

Verify:

```bash id="e3yw4h"
sudo lvdisplay
```

---

## 🔹 Step 5: Create filesystem

```bash id="0u4yib"
sudo mkfs.ext4 /dev/my_vg/my_lv
```

---

## 🔹 Step 6: Create mount point

```bash id="h31xxc"
sudo mkdir /data
```

---

## 🔹 Step 7: Mount the volume

```bash id="pjm1tz"
sudo mount /dev/my_vg/my_lv /data
```

Verify:

```bash id="7k1jbh"
df -h
```

---

## 🔹 Step 8: Permanent mount

```bash id="bkf6x0"
sudo blkid
```

Edit:

```bash id="avdaxu"
sudo vi /etc/fstab
```

Add:

```id="b6r62u"
UUID=<uuid>  /data  ext4  defaults  0  2
```

Apply:

```bash id="1j2hz5"
sudo mount -a
```

---

# 🔄 **Bonus: Extend Logical Volume**

## Increase size

```bash id="b0cnmo"
sudo lvextend -L +2G /dev/my_vg/my_lv
```

## Resize filesystem

```bash id="jb7nfi"
sudo resize2fs /dev/my_vg/my_lv
```

---

# 🎯 **Real-world use case**

* Combine multiple EBS volumes into one VG
* Dynamically increase storage without downtime
* Used heavily in production systems

---
