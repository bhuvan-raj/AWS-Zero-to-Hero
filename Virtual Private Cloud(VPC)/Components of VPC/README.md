
# AWS VPC – Components 

## 1. What is an AWS VPC?

An **Amazon Virtual Private Cloud (VPC)** is a logically isolated virtual network in AWS where you can launch AWS resources such as EC2 instances, RDS, and load balancers.
You control:

* IP addressing
* Subnets
* Routing
* Network gateways
* Security boundaries

A VPC closely resembles a traditional on-premises network but with cloud scalability.

---

## 2. Subnet in AWS VPC
<img src="https://github.com/bhuvan-raj/AWS-Zero-to-Hero/blob/main/assets/subnet.png" alt="Banner" />


### What is a Subnet?

A **subnet** is a range of IP addresses within a VPC.
Subnets allow you to **divide your VPC network into smaller segments**.

Key points:

* A subnet is **always created inside one Availability Zone (AZ)**
* You cannot span a subnet across multiple AZs
* Each subnet uses a CIDR block derived from the VPC CIDR

---

### Reserved IP Addresses in a Subnet

AWS reserves **5 IP addresses** in every subnet:

* Network address (first IP)
* VPC router
* DNS
* Future use
* Broadcast address (last IP)

Example:
CIDR: `10.0.1.0/24`
Usable IPs: `251`

---

### Types of Subnets

#### 1️⃣ Public Subnet

A subnet is **public** if:

* Its route table has a route to an **Internet Gateway (IGW)**

Used for:

* Web servers
* Bastion hosts
* Load balancers

Example route:

```
0.0.0.0/0 → Internet Gateway
```

---

#### 2️⃣ Private Subnet

A subnet is **private** if:

* It does NOT have a direct route to the Internet Gateway

Used for:

* Application servers
* Databases
* Internal services

Internet access (outbound only) is possible via **NAT Gateway**.

---

### How to Create a Subnet (AWS Console)

1. Go to **VPC Dashboard**
2. Click **Subnets → Create subnet**
3. Select:

   * VPC
   * Availability Zone
   * IPv4 CIDR block (example: `10.0.1.0/24`)
4. Click **Create subnet**

---

## 3. Route Table
<img src="https://github.com/bhuvan-raj/AWS-Zero-to-Hero/blob/main/assets/rt.png" alt="Banner" />

### What is a Route Table?

A **route table** defines **how traffic is routed** within the VPC and outside the VPC.

Each subnet must be associated with **one route table**.

---

### Route Table Components

A route table contains:

* **Destination** – IP range (CIDR)
* **Target** – Where the traffic should go

Example:

```
Destination     Target
10.0.0.0/16     local
0.0.0.0/0       igw-xxxx
```
---

### Default Route Table

Every VPC has a **default route table**:

* Contains only the `local` route
* Used when no custom route table is associated

---

### How to Create a Route Table

1. Go to **VPC → Route Tables**
2. Click **Create route table**
3. Select the VPC
4. Add routes (IGW or NAT)
5. Associate the route table with subnets

---

## 4. Internet Gateway (IGW)

### What is an Internet Gateway?

An **Internet Gateway** enables **two-way communication** between:

* Resources in a VPC
* The public internet

Required for:

* Public subnets
* EC2 instances with public IPs

---

### Key Characteristics

* Horizontally scaled
* Highly available
* One IGW per VPC
* Supports IPv4 and IPv6

---

### How Internet Access Works

For internet access:

1. Subnet route table → IGW
2. EC2 has a public IP / Elastic IP
3. Security Group allows traffic

---

### How to Create an Internet Gateway

1. Go to **VPC → Internet Gateways**
2. Click **Create internet gateway**
3. Attach it to a VPC
4. Add IGW route to subnet route table

---

## 5. NAT Gateway (In-Depth)

### What is a NAT Gateway?

A **NAT Gateway** allows **private subnet instances** to:

* Access the internet (outbound only)
* Block inbound internet traffic

Used mainly for:

* Software updates
* Accessing external APIs

---

### Key Characteristics

* Managed AWS service
* Highly available within an AZ
* Requires Elastic IP
* Must be deployed in a **public subnet**

---

### NAT Gateway Traffic Flow

```
Private Subnet → NAT Gateway → Internet Gateway → Internet
```

Inbound traffic from the internet is **not allowed**.

---

### How to Create a NAT Gateway

1. Create an **Elastic IP**
2. Go to **VPC → NAT Gateways**
3. Click **Create NAT Gateway**
4. Select:

   * Public Subnet
   * Elastic IP
5. Update private subnet route table:

```
0.0.0.0/0 → NAT Gateway
```

---

## 6. Summary Architecture Example

| Component        | Purpose                               |
| ---------------- | ------------------------------------- |
| VPC              | Network boundary                      |
| Public Subnet    | Internet-facing resources             |
| Private Subnet   | Backend resources                     |
| Route Table      | Traffic routing                       |
| Internet Gateway | Public internet access                |
| NAT Gateway      | Outbound internet for private subnets |

---

