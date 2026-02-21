
#  What is VPC Peering?

Amazon Web Services VPC Peering is a networking connection between two VPCs that enables them to route traffic privately using private IP addresses.

It works:

* Within same region
* Across regions (Inter-Region Peering)
* Same account or different accounts

No traffic goes through the public internet.

---

# 2️⃣ Architecture Overview

![Image](https://docs.aws.amazon.com/images/prescriptive-guidance/latest/integrate-third-party-services/images/p2_vpc-peering.png)

Example:

VPC-A (10.0.0.0/16)
VPC-B (192.168.0.0/16)

After peering:

* Add route in VPC-A → 192.168.0.0/16 → peering connection
* Add route in VPC-B → 10.0.0.0/16 → peering connection

Now instances communicate privately.

---

#  Key Requirements

### ✅ 1. CIDR must NOT overlap

Example:

* VPC1: 10.0.0.0/16
* VPC2: 10.0.0.0/16 ❌ (Not allowed)

### ✅ 2. Route tables must be updated manually

### ✅ 3. Security groups must allow traffic

Peering does NOT automatically update:

* Route tables
* Security groups
* NACLs

---

#  How VPC Peering Works (Step-by-Step)

### Step 1: Create Peering Request

From VPC-A → Create Peering → Select VPC-B

### Step 2: Accept Peering Request

Other account/region must accept it.

### Step 3: Update Route Tables

Add routes:

| Destination    | Target    |
| -------------- | --------- |
| 192.168.0.0/16 | pcx-xxxxx |

### Step 4: Update Security Groups

Allow inbound from peer CIDR.

---

#  Types of VPC Peering

## 🔹 Same Region Peering

Low latency, high bandwidth.

## 🔹 Inter-Region Peering

Private communication across regions.
Traffic stays within AWS backbone.

---

#  Important Characteristics

### ❌ No Transitive Peering

If:

* VPC-A ↔ VPC-B
* VPC-B ↔ VPC-C

VPC-A CANNOT talk to VPC-C.

For hub-spoke model, use:

* AWS Transit Gateway

---

### ❌ No Edge-to-Edge Routing

You cannot use peering to:

* Access Internet Gateway of peer
* Access NAT Gateway of peer
* Access VPN of peer
* Access Direct Connect of peer

Each VPC needs its own gateways.

---

#  DNS Resolution in Peering

By default:

* Private DNS does NOT resolve across VPCs.

You must enable:

* DNS resolution from accepter side
* DNS resolution from requester side

Then private hosted zones work.

---

#  Security Considerations

Even if peering exists:

* Security Groups still control traffic
* NACL still applies
* Routing must be correct

Best practice:

* Allow only specific ports
* Avoid wide CIDR allow rules

---

#  Performance & Cost

### Performance

* Uses AWS backbone
* High bandwidth
* Low latency

### Cost

* Same region: charged for data transfer
* Inter-region: higher data transfer cost

No hourly charge for peering itself.

--- 
# Real-World Use Cases

### ✅ Microservices isolation

Frontend VPC ↔ Backend VPC

### ✅ Shared services VPC

App VPC ↔ Logging VPC

### ✅ Cross-account communication

Dev account ↔ Shared infra account

### ✅ Multi-region architecture

Primary region ↔ DR region
---

#  Limitations

* No transitive routing
* No overlapping CIDR
* Manual route updates
* Scaling complexity
* Cannot reference SG across regions (only same region)

# 🧪 LAB: VPC Peering (Same Region, Same Account)

## 🎯 Objective

Establish private communication between two VPCs using VPC Peering.

### Network Plan

| Resource | CIDR           |
| -------- | -------------- |
| VPC-A    | 10.0.0.0/16    |
| Subnet-A | 10.0.1.0/24    |
| VPC-B    | 192.168.0.0/16 |
| Subnet-B | 192.168.1.0/24 |

Region: Same region (example: ap-south-1)

---

# 2️⃣ Step 1 – Create VPC-A

1. Go to **VPC → Create VPC**
2. Name: VPC-A
3. CIDR: 10.0.0.0/16
4. Create Subnet:

   * CIDR: 10.0.1.0/24
5. Create Internet Gateway (optional for SSH)
6. Attach IGW to VPC-A
7. Update Route Table:

   * 0.0.0.0/0 → IGW

---

# 3️⃣ Step 2 – Launch EC2 in VPC-A

Use:

* Amazon Web Services EC2
* Amazon Linux AMI
* Subnet: 10.0.1.0/24
* Enable auto-assign public IP (for SSH testing)

Security Group:

* Allow SSH (22) from your IP
* Allow ICMP from 192.168.0.0/16

---

# 4️⃣ Step 3 – Create VPC-B

1. Create VPC

   * Name: VPC-B
   * CIDR: 192.168.0.0/16
2. Create Subnet:

   * CIDR: 192.168.1.0/24
3. Create Internet Gateway (optional)
4. Attach IGW
5. Add route: 0.0.0.0/0 → IGW

---

# 5️⃣ Step 4 – Launch EC2 in VPC-B

* Same AMI
* Subnet: 192.168.1.0/24
* Public IP enabled

Security Group:

* Allow SSH from your IP
* Allow ICMP from 10.0.0.0/16

---

# 6️⃣ Step 5 – Create VPC Peering Connection

1. Go to **VPC → Peering Connections**
2. Click Create Peering
3. Requester VPC → VPC-A
4. Accepter VPC → VPC-B
5. Create

Now:

* Select Peering
* Click **Actions → Accept Request**

Status should become **Active**

---

# 7️⃣ Step 6 – Update Route Tables (Very Important)

### In VPC-A Route Table

Add:

| Destination    | Target   |
| -------------- | -------- |
| 192.168.0.0/16 | pcx-xxxx |

---

### In VPC-B Route Table

Add:

| Destination | Target   |
| ----------- | -------- |
| 10.0.0.0/16 | pcx-xxxx |

Without this step, traffic will not flow.

---

# 8️⃣ Step 7 – Verify Security Groups

Ensure:

VPC-A EC2:

* Inbound → ICMP → Source: 192.168.0.0/16

VPC-B EC2:

* Inbound → ICMP → Source: 10.0.0.0/16

---

# 9️⃣ Step 8 – Test Connectivity

SSH into EC2-A.

Run:

```bash
ping <Private-IP-of-EC2-B>
```

Expected:
Ping successful.

Then test reverse direction.

---

# 🔎 Troubleshooting Checklist

If ping fails:

✔ Check route table
✔ Check security group
✔ Check NACL
✔ Confirm peering status = Active
✔ Confirm CIDR does not overlap

---

# 📌 Important Observations

1. No transitive routing
2. No sharing of Internet Gateway
3. Manual route configuration required
4. CIDR blocks must not overlap

---

# 🔬 Optional Advanced Testing

### Test Port Connectivity

From EC2-A:

```bash
nc -zv <Private-IP> 22
```

### Enable VPC Flow Logs

To monitor traffic:
VPC → Flow Logs → Create Flow Log

---
