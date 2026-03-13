# Elastic Network Interface (ENI) in AWS EC2

An **Elastic Network Interface (ENI)** is a virtual network interface that you can attach to an EC2 instance within a Virtual Private Cloud (VPC).

It acts as a **network card (NIC)** for your EC2 instance and allows the instance to communicate with other resources inside or outside the VPC.

---

# 1. What is an ENI?

An Elastic Network Interface is a **virtual network interface attached to an EC2 instance**.

It enables communication between:

* EC2 instances
* Other services in the VPC
* The internet (if configured)

Each EC2 instance has **at least one network interface called the Primary Network Interface (eth0).**

Additional interfaces can also be attached depending on the instance type.

---

# 2. Components of an ENI

An ENI contains several networking attributes.

| Component             | Description                                 |
| --------------------- | ------------------------------------------- |
| Private IP Address    | Primary private IP assigned to the ENI      |
| Secondary Private IPs | Additional private IPs that can be assigned |
| Public IP             | Optional public IP mapped to the private IP |
| Elastic IP            | Static public IP that can be associated     |
| Security Groups       | Firewall rules controlling traffic          |
| MAC Address           | Unique hardware identifier                  |
| Subnet                | The subnet where the ENI resides            |

---

# 3. Types of Network Interfaces

## Primary Network Interface

The primary network interface is automatically created when the EC2 instance is launched.

Characteristics:

* Cannot be detached from the instance
* Located in the same subnet as the instance
* Known as **eth0**

Example:

```bash
eth0
```

---

## Secondary Network Interfaces

You can attach **additional ENIs** to an EC2 instance.

Characteristics:

* Can be attached or detached
* Used for advanced networking configurations
* Each interface gets its own private IPs and security groups

Example:

```bash
eth1
eth2
```

---

# 4. Key Features of ENI

## Multiple IP Addresses

An ENI can have multiple private IP addresses.

Example:

```
Primary IP: 10.0.1.10
Secondary IP: 10.0.1.11
Secondary IP: 10.0.1.12
```

This allows one instance to host multiple services.

---

## Elastic IP Association

You can associate an **Elastic IP (EIP)** with an ENI to provide a **static public IP address**.

Benefits:

* Persistent public address
* Survives instance stop/start

---

## Security Group Assignment

Each ENI can have **one or more security groups** attached.

This allows **fine-grained traffic control** at the network interface level.

---

# 5. ENI Limits

The number of ENIs that can be attached depends on the **EC2 instance type**.

Example:

| Instance Type | Max ENIs |
| ------------- | -------- |
| t2.micro      | 2        |
| t3.medium     | 3        |
| m5.large      | 3        |
| c5.2xlarge    | 4        |

Each ENI can also support **multiple IP addresses**.

---

# 6. Use Cases of ENI

## High Availability

An ENI can be moved from one instance to another quickly.

This is useful for:

* Failover systems
* Disaster recovery setups

Example scenario:

```
Primary Server (Failure)
      ↓
Detach ENI
      ↓
Attach to Backup Server
```

---

## Network Appliances

ENIs are used in:

* Firewalls
* Load balancers
* NAT instances
* Security monitoring systems

These applications often require **multiple network interfaces**.

---

## Multi-Homed Instances

An EC2 instance can have multiple ENIs attached to different subnets.

Example:

```
eth0 → Public subnet
eth1 → Private subnet
```

This is useful for **network routing and isolation**.

---

# 7. Example Architecture

```
Internet
   │
Internet Gateway
   │
Public Subnet
   │
EC2 Instance
   │
ENI (eth0)
   ├── Private IP
   ├── Public IP
   └── Security Group
```

---



#  Important Notes

* ENIs **must be in the same Availability Zone as the instance**
* Primary ENI cannot be detached
* Secondary ENIs can be attached/detached
* ENIs maintain their IP addresses when moved

---

# Summary

| Feature         | Description                             |
| --------------- | --------------------------------------- |
| ENI             | Virtual network card for EC2            |
| Primary ENI     | Default interface created with instance |
| Secondary ENI   | Additional attachable interface         |
| IP Addresses    | Supports multiple private IPs           |
| Security Groups | Applied at ENI level                    |
| Elastic IP      | Can be attached for static public IP    |

---

# Conclusion

Elastic Network Interfaces provide **flexible networking capabilities** for EC2 instances. They enable advanced network configurations such as **multiple IP addresses, failover mechanisms, and multi-network architectures**, making them a fundamental component of AWS networking.
