
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


---
