# What is AWS VPC VPN?

In Amazon Web Services, a **Site-to-Site VPN** connects your on-premises data center to a VPC securely using IPsec encryption.

It allows:

* Hybrid cloud architecture
* Secure data transfer
* Private connectivity without Direct Connect

---

# Types of AWS VPN

## 🔹 1. Site-to-Site VPN

Connects on-prem network ↔ AWS VPC.

## 🔹 2. Client VPN

Allows individual users to connect securely to AWS.

---

# Core Components of Site-to-Site VPN

![Image](https://docs.aws.amazon.com/images/whitepapers/latest/aws-vpc-connectivity-options/images/redundant-aws-site-to-site-vpn-connections.png)

### Components:

##  Virtual Private Gateway (VGW)

AWS-side VPN endpoint attached to VPC.

##  Customer Gateway (CGW)

Represents your on-premises router/firewall.

* Public IP required
* Static or dynamic routing

##  VPN Connection

Logical connection between VGW and CGW.
Includes:

* Two IPsec tunnels (High availability)

---

#  How It Works (Technical Flow)

1. IPsec tunnel established between CGW and VGW
2. IKE (Internet Key Exchange) negotiates keys
3. Two tunnels created (Active/Passive)
4. Routing exchanges happen
5. Encrypted traffic flows over internet

Encryption:

* AES
* SHA
* DH Groups

---

#  Routing Types

## 🔹 Static Routing

You manually define destination CIDRs.

Example:

* On-prem: 172.16.0.0/16
* AWS: 10.0.0.0/16

## 🔹 Dynamic Routing (Recommended)

Uses BGP (Border Gateway Protocol).

Advantages:

* Automatic route updates
* Failover support
* Better scalability

---

# High Availability

Each VPN connection creates:

* 2 IPsec tunnels
* Two AWS endpoints (different Availability Zones)

If one tunnel fails:
Traffic automatically shifts to second.

---

# Implementation Steps (High-Level)

### Step 1: Create VPC

CIDR: 10.0.0.0/16

### Step 2: Create Virtual Private Gateway

Attach to VPC.

### Step 3: Create Customer Gateway

Provide:

* On-prem public IP
* BGP ASN (if dynamic routing)

### Step 4: Create VPN Connection

Select:

* VGW
* CGW
* Routing type

### Step 5: Update Route Tables

Add route:

* 172.16.0.0/16 → VGW

### Step 6: Configure On-Prem Router

Download AWS VPN configuration.
Apply IPsec settings.

---

# 8️⃣ VPN vs Direct Connect

| Feature       | VPN     | Direct Connect |
| ------------- | ------- | -------------- |
| Uses Internet | Yes     | No             |
| Encryption    | Yes     | Optional       |
| Cost          | Low     | High           |
| Latency       | Higher  | Lower          |
| Bandwidth     | Limited | Dedicated      |

Often combined:
Direct Connect + VPN backup.

---

#  VPN with Transit Gateway

Instead of VGW, you can use:

* AWS Transit Gateway

Advantages:

* Connect multiple VPCs
* Centralized routing
* Scalable architecture

Best for large enterprises.

---

# Security Considerations

✔ Uses IPsec encryption
✔ Supports Perfect Forward Secrecy
✔ Traffic encrypted end-to-end
✔ Supports strong cipher suites

Best Practices:

* Use dynamic routing
* Monitor tunnel health
* Enable CloudWatch logs
* Use redundant CGW devices

---

#  Limitations

* Internet-based (latency variability)
* Throughput limits (~1.25 Gbps per tunnel)
* Requires public IP on customer side
* Dependent on internet stability

---

#  Monitoring & Troubleshooting

Check:

* Tunnel status (UP/DOWN)
* BGP status
* CloudWatch metrics
* Route table propagation

Common Issues:

* Incorrect pre-shared key
* Firewall blocking UDP 500/4500
* BGP ASN mismatch
* Overlapping CIDR

---

#  Real-World Use Cases

✅ Hybrid cloud migration
✅ Disaster recovery
✅ Secure backend connectivity
✅ Multi-site connectivity

---

#  Interview Explanation (Concise Version)

AWS VPC VPN is an IPsec-based secure connection between an on-premises network and a VPC. It uses a Virtual Private Gateway and a Customer Gateway, supports static or dynamic routing with BGP, and creates two redundant encrypted tunnels for high availability. It is cost-effective but internet-dependent, unlike Direct Connect.

---
