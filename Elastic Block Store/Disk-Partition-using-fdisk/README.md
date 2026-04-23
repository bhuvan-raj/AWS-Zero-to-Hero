
# **Lab Note: Disk Partitioning of an EBS Volume using fdisk**

## **Objective**

To create an EBS volume, attach it to an EC2 instance, and perform disk partitioning using `fdisk`.

---

## **Prerequisites**

* AWS account
* Running EC2 instance (Linux)
* SSH access to the instance

---

## **Step 1: Create an EBS Volume**

1. Go to **AWS Console → EC2 → Elastic Block Store → Volumes**
2. Click **Create Volume**
3. Configure:

   * Volume type: `gp3` (recommended)
   * Size: e.g., `10 GiB`
   * Availability Zone: **Same as EC2 instance**
4. Click **Create Volume**

---

## **Step 2: Attach EBS Volume to EC2**

1. Select the created volume
2. Click **Actions → Attach Volume**
3. Choose your EC2 instance
4. Set device name (example):

   ```
   /dev/xvdbb
   ```
5. Click **Attach**

---

## **Step 3: Connect to EC2 Instance**

```bash
ssh -i your-key.pem ec2-user@your-public-ip
```

---

## **Step 4: Verify Attached Disk**

Check available disks:

```bash
lsblk
```

or

```bash
sudo fdisk -l
```

You should see something like:

```
/dev/xvdbb   10G
```

---

## **Step 5: Partition the Disk using fdisk**

Start fdisk:

```bash
sudo fdisk /dev/xvdbb
```

### Inside fdisk:

1. Create new partition:

   ```
   n
   ```

2. Choose partition type:

   ```
   p   (primary)
   ```

3. Partition number:

   ```
   1
   ```

4. Press Enter for default first sector

5. Press Enter for default last sector

6. Save changes:

   ```
   w
   ```

---

## **Step 6: Verify Partition**

```bash
lsblk
```

Output will show:

```
/dev/xvdbb1
```

---

## **Step 7: Format the Partition**

Format using ext4:

```bash
sudo mkfs.ext4 /dev/xvdbb1
```

---

## **Step 8: Create Mount Point**

```bash
sudo mkdir /data
```

---

## **Step 9: Mount the Partition**

```bash
sudo mount /dev/xvdbb1 /data
```

Verify:

```bash
df -h
```

---

## **Step 10: Permanent Mount (Optional)**

Get UUID:

```bash
sudo blkid
```

Edit fstab:

```bash
sudo vi /etc/fstab
```

Add entry:

```
UUID=xxxx-xxxx  /data  ext4  defaults,nofail  0  2
```

Save and test:

```bash
sudo mount -a
```

---

## **Result**

* EBS volume successfully created and attached
* Disk partition created using `fdisk`
* File system created and mounted

---

## **Notes**

* Device name may appear as `/dev/nvme1n1` in newer instances (NVMe-based)
* Always ensure correct device before partitioning to avoid data loss

