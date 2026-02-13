
# Bastion Hosting — Comprehensive Notes

## 1. What Is a Bastion Host?

A **bastion host** is a specially configured server designed to withstand attacks and serve as a secure entry point into a private network. It is hardened, monitored, and exposed to untrusted networks (like the public internet) so that internal systems remain isolated and protected.

The term *bastion* comes from military usage — a fortified position designed to defend against direct attacks. In networking, it plays the same role: a secure access gateway.

---

## **2. Why Use a Bastion Host?**

The primary goals of using a bastion host are:

✔ **Secure Access:** Provides controlled administrative access (SSH/RDP) into internal systems
✔ **Network Isolation:** Keeps direct access to private resources restricted
✔ **Threat Surface Reduction:** Limits entry points exposed to public networks
✔ **Audit & Accountability:** Centralizes logging of access events

Without a bastion host, administrative access might be opened directly to each host in a private network, significantly increasing risk.

---

## **3. Typical Use Cases**

| Scenario                           | Role of Bastion Host                           |
| ---------------------------------- | ---------------------------------------------- |
| Managing private servers (SSH/RDP) | Acts as the only externally accessible host    |
| Regulatory compliance              | Records and enforces access auditing           |
| Zero Trust Architecture            | Enforces least-privilege access                |
| Hybrid cloud environments          | Provides secure bridge between on-prem & cloud |

---

## **4. How It Works**

### **Network Architecture**

A typical architecture using a bastion host:

```
Internet
   │
Public Subnet (Bastion Host)
   │
Private Subnet (App Servers/Databases)
```

Only the bastion host has a **public IP**. Internal servers reside in **private subnets** with no direct internet exposure.

### **Access Flow**

1. Administrator connects to the bastion host (SSH/RDP).
2. From the bastion, a second connection is made to internal servers.
3. Internal hosts are never exposed directly to the internet.

---

## **5. Security Hardening Best Practices**

A bastion host must be **hardened** to resist attacks:

### **Authentication**

* Use **SSH key pairs**, not passwords
* Prefer **MFA (Multi-Factor Authentication)**
* Integrate with identity providers (LDAP/Active Directory)

### **Network Controls**

* Allow inbound only from trusted IP ranges
* Restrict outbound so that the bastion can only reach internal systems it needs
* Use security groups/firewalls

### **Operating System Hardening**

* Disable unused services/ports
* Keep OS and packages updated
* Enforce strong password policies (if any)

### **Logging & Monitoring**

* Enable verbose auth logs
* Send logs to a central system (SIEM, CloudWatch, Splunk)
* Alert on suspicious activity (multiple failed logins, unusual times)

### **Session Management**

* Use jump host session recording
* Restrict session timeouts
* Periodically rotate SSH keys



# 🧪 LAB: Check Internet Connectivity of a Private EC2 Using Bastion Host (AWS)

## 🏗️ Step 1: Create VPC Architecture

### 1️⃣ Create VPC

Go to **VPC → Create VPC**

* Name: `bastion-lab-vpc`
* IPv4 CIDR: `10.0.0.0/16`
* Tenancy: Default

---

## 2️⃣ Create Subnets

### Public Subnet

* Name: `public-subnet`
* AZ: Any (e.g., ap-south-1a)
* CIDR: `10.0.1.0/24`
* Enable **Auto-assign Public IP**

### Private Subnet

* Name: `private-subnet`
* AZ: Same AZ
* CIDR: `10.0.2.0/24`

---
## Create Private and Public Routetables

- Route tables - create - Private RT - Select VPC - Create - Associate the RT with appropriate subnet
- Route tables - create - Public RT - Select VPC - Create - Associate the RT with appropriate subnet


## 3️⃣ Create Internet Gateway (IGW)

Go to **Internet Gateway → Create**

* Name: `bastion-igw`
* Attach to `bastion-lab-vpc`

---

## 4️⃣ Create NAT Gateway

Go to **NAT Gateway → Create**

* Name: `bastion-nat`
* Type- Automatic

This allows private instances to access the internet.

---

## 5️⃣ Configure Route Tables

### Public Route Table

* Create: `public-rt`
* Add route:

  * Destination: `0.0.0.0/0`
  * Target: Internet Gateway
    

### Private Route Table

* Create: `private-rt`
* Add route:

  * Destination: `0.0.0.0/0`
  * Target: NAT Gateway

---


# 🖥️ Step 2: Launch EC2 Instances

## 1️⃣ Launch Bastion Host

* AMI: Amazon Linux
* Instance Type: t3.micro (Free tier)
* Networking - VPC - bastion-lab-vpc
* Subnet: `public-subnet`
* Auto-assign Public IP: Enabled
* Key Pair: Create or use existing

Name: `bastion-host`

---

## 2️⃣ Launch Private EC2

* AMI: Amazon Linux
* Instance Type: t3.micro
* Networking - VPC - bastion-lab-vpc
* Subnet: `private-subnet`
* Auto-assign Public IP: Disabled
* Same Key Pair

Name: `private-server`

---

# 🔑 Step 4: Connect to Bastion Host

From your local machine:

```bash
ssh -i your-key.pem ec2-user@<Bastion-Public-IP>
```

---

# 🔁 Step 5: Connect to Private EC2 from Bastion

Inside bastion:

### Create a Private key
```
nano my-key.pem
```
- copy paste the contents of private key from your local machine
```
Ctl + x , y and enter
```
### give execute permission to the key

```bash
chmod 700 your-key.pem
```

```bash
ssh -i your-key.pem ec2-user@<Private-EC2-Private-IP>
```

---

# 🌍 Step 6: Check Internet Connectivity from Private EC2

Now from the **private instance**, test outbound connectivity:

### Test 1: Ping Google DNS

```bash
ping 8.8.8.8
```

### Test 2: Curl Google

```bash
curl https://google.com
```

### Test 3: Install Package

```bash
sudo yum update -y
```

If update works → NAT Gateway is functioning.

---

# 🔍 How It Works (Conceptually)

1. Private EC2 sends traffic to `0.0.0.0/0`
2. Route table forwards to NAT Gateway
3. NAT Gateway sends traffic to Internet Gateway
4. Response comes back through NAT
5. Bastion is NOT used for internet — only for SSH access

---

# 📌 Important Observations

| Component        | Role                                     |
| ---------------- | ---------------------------------------- |
| Internet Gateway | Enables public subnet internet           |
| NAT Gateway      | Enables private subnet outbound internet |
| Bastion Host     | Secure SSH entry point                   |
| Route Tables     | Control traffic flow                     |

---

# 🛑 Common Troubleshooting

If internet does NOT work:

✔ Check private route table → 0.0.0.0/0 → NAT
✔ Check NAT is in public subnet
✔ Ensure public subnet has IGW route
✔ Ensure security group allows outbound
✔ Check NACL rules

---

# 🧠 Interview-Level Understanding

If asked:

> “How does private EC2 access the internet?”

Answer:

> Private EC2 routes outbound traffic to a NAT Gateway in a public subnet, which forwards traffic via an Internet Gateway, allowing outbound-only internet access without exposing the instance publicly.

---
