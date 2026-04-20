# EBS Permanet Mounting

## 🔹 1. Attach EBS Volume to EC2

* Go to AWS Console → EC2 → Volumes
* Select volume → **Attach Volume**
* Choose your EC2 instance
* Device name example: `/dev/xvdf`

---

## 🔹 2. Connect to Instance

```bash
ssh ec2-user@your-instance-ip
```

(or `ubuntu@...` depending on AMI)

---

## 🔹 3. Verify the Volume

Check if the disk is visible:

```bash
lsblk
```

You’ll see something like:

```
xvda   (root disk)
xvdf   (new EBS volume)
```

👉 If not visible:

```bash
sudo fdisk -l
```

---

## 🔹 4. Create Filesystem (First Time Only)

⚠️ Do this only if the volume is new (no data)

```bash
sudo mkfs -t ext4 /dev/xvdf
```

---

## 🔹 5. Create Mount Point

```bash
sudo mkdir /data
```

---

## 🔹 6. Mount the Volume (Temporary)

```bash
sudo mount /dev/xvdf /data
```

Verify:

```bash
df -h
```

---

## 🔹 7. Get UUID (Important for Permanent Mount)

Never use `/dev/xvdf` directly in `/etc/fstab` because device names can change.

```bash
sudo blkid
```

Example output:

```
/dev/xvdf: UUID="abcd-1234-xyz" TYPE="ext4"
```

---

## 🔹 8. Configure Permanent Mount (/etc/fstab)

Edit fstab:

```bash
sudo vi /etc/fstab
```

Add this line:

```
UUID=abcd-1234-xyz  /data  ext4  defaults,nofail  0  2
```

### Explanation:

* `UUID` → unique disk ID
* `/data` → mount point
* `ext4` → filesystem
* `defaults` → standard options
* `nofail` → prevents boot failure if volume missing
* `0` → no dump
* `2` → filesystem check order

---

## 🔹 9. Test Before Reboot (Critical Step)

```bash
sudo mount -a
```

👉 If no error → configuration is correct
👉 If error → fix before reboot (otherwise instance may not boot)

---

## 🔹 10. Reboot and Verify

```bash
sudo reboot
```

After login:

```bash
df -h
```

Confirm `/data` is mounted.

---

# 🔹 Common Real-World Issues & Fixes

## ❌ Device name changes

* `/dev/xvdf` may become `/dev/nvme1n1` (especially in Nitro instances)
* ✅ Always use UUID

---

## ❌ Wrong fstab entry → instance won’t boot

* Fix using EC2 **rescue instance**
* Or use `nofail` option to avoid crash

---

## ❌ Permission issues

```bash
sudo chown ec2-user:ec2-user /data
```

---

## ❌ Already has filesystem?

Check before formatting:

```bash
sudo file -s /dev/xvdf
```

---

# 🔹 Bonus: Mount Without UUID (Not Recommended)

```bash
/dev/xvdf  /data  ext4  defaults  0  2
```

👉 Risky because device names can change

---

# 🔹 Interview Tips (Important)

If asked:

👉 **Best practice**

* Use UUID in `/etc/fstab`
* Test using `mount -a`
* Use `nofail` to prevent boot issues
