<img src="https://github.com/bhuvan-raj/AWS-Zero-to-Hero/blob/main/assets/z2h.jpeg" alt="Banner" />

<div align="center">

![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner%20to%20Advanced-purple?style=for-the-badge)
![IAM](https://img.shields.io/badge/IAM-Security-red?style=for-the-badge&logo=amazon-aws&logoColor=white)
![S3](https://img.shields.io/badge/S3-Storage-569A31?style=for-the-badge&logo=amazon-s3&logoColor=white)
![VPC](https://img.shields.io/badge/VPC-Networking-blue?style=for-the-badge)
![Contributions](https://img.shields.io/badge/Contributions-Welcome-brightgreen?style=for-the-badge)

# AWS — Zero to Hero

### ☁️ From cloud fundamentals to advanced AWS architecture — comprehensive explanations, hands-on labs, and real-world best practices.

</div>

---

## 📖 About This Repository

**AWS Zero to Hero** is a structured, project-based learning repository designed to take you from understanding the basics of cloud computing all the way through deploying secure, scalable, and highly available architectures on Amazon Web Services.

Every section is organized into its own folder with a dedicated `README.md` covering in-depth explanations, diagrams, AWS Console walkthroughs, and hands-on labs. The content progresses logically — from foundational cloud concepts to advanced AWS networking and security.

By the end of this course, you will be able to:

- ✅ Explain cloud computing concepts, deployment models, and service models
- ✅ Navigate the AWS global infrastructure — Regions, AZs, and Edge Locations
- ✅ Implement fine-grained access control using IAM users, roles, policies, and Organizations
- ✅ Store, secure, and manage data at scale with Amazon S3
- ✅ Design isolated, secure cloud networks using AWS VPC
- ✅ Connect and protect VPCs using Peering, Transit Gateway, Endpoints, and DNS Firewall

---

## 📚 Table of Contents

## 1. Introduction to Cloud Computing

This section introduces the fundamentals of cloud computing — what it is, why it exists, and how it differs from traditional on-premises IT infrastructure. It covers key characteristics, business benefits, and real-world use cases that form the foundation for understanding modern cloud platforms like AWS.

📂 **[Explore → Introduction to Cloud Computing](./Introduction%20to%20cloud%20computing/)**

---

## 2. Cloud Deployment and Service Models

This section explains how cloud environments are architected and consumed. It covers cloud **deployment models** — Public, Private, Hybrid, and Community — alongside cloud **service models** — IaaS, PaaS, and SaaS — helping you understand the architectural and usage choices available in cloud computing.

📂 **[Explore → Cloud Deployment and Service Models](./Cloud%20Deployment%20and%20Service%20Models/)**

---

## 3. Introduction to AWS

This section provides an overview of Amazon Web Services — its global infrastructure, how AWS organizes compute and storage capacity across Regions and Availability Zones, and a high-level introduction to the essential AWS services that power modern cloud applications.

📂 **[Explore → Introduction to AWS](./Introduction%20to%20AWS/)**

---

## 4. AWS Identity and Access Management (IAM)

This section provides a detailed understanding of AWS IAM — how authentication and authorization work in AWS, and how to implement the principle of least privilege at every layer of your environment.

📂 **[Explore → AWS Identity and Access Management](./AWS%20Identity%20and%20Access%20Management/)**

| # | Sub-Topic | Description |
|---|-----------|-------------|
| 4.1 | [Introduction to AWS IAM](./AWS%20Identity%20and%20Access%20Management/Introduction%20to%20aws%20iam/) | IAM overview, users, groups, and foundational concepts |
| 4.2 | [Roles and Policies in IAM](./AWS%20Identity%20and%20Access%20Management/Roles%20and%20Policies%20in%20IAM/) | IAM roles, managed vs inline policies, and policy structure |
| 4.3 | [Policy Evaluation and Cross Account Access](./AWS%20Identity%20and%20Access%20Management/Policy%20Evaluation%20&%20Cross%20Account%20Access/) | How AWS evaluates policies and enables cross-account resource access |
| 4.4 | [AWS Organizations, SCPs and Identity Center](./AWS%20Identity%20and%20Access%20Management/AWS%20Organisation,%20SCPs%20and%20Identity%20Center/) | Managing multiple accounts, Service Control Policies, and AWS Identity Center |
| 4.5 | [Single Sign-On (SSO)](./AWS%20Identity%20and%20Access%20Management/Single%20Sign-On/) | Centralized access to AWS accounts and business applications |

---

## 5. AWS Simple Storage Service (S3)

This section provides a detailed understanding of Amazon S3 — how to store, manage, secure, and retrieve data at scale. It covers everything from core concepts to advanced features like encryption, replication, lifecycle policies, and attribute-based access control.

📂 **[Explore → AWS Simple Storage Service](./AWS%20Simple%20Storage%20Service(S3)/)**

| # | Sub-Topic | Description |
|---|-----------|-------------|
| 5.1 | [Cloud Storage & S3 Fundamentals](./AWS%20Simple%20Storage%20Service(S3)/Cloud%20Storage%20&%20S3%20Fundamentals/) | What S3 is, core storage concepts, durability, and availability |
| 5.2 | [S3 Bucket Types & Architecture](./AWS%20Simple%20Storage%20Service(S3)/S3%20Bucket%20Types%20&%20Architecture/) | General-purpose vs directory buckets, flat namespace architecture |
| 5.3 | [Objects, Storage Classes & Cost Optimization](./AWS%20Simple%20Storage%20Service(S3)/Objects,%20Storage%20Classes%20&%20Cost%20Optimization/) | Object structure, versioning, storage tiers, and cost management |
| 5.4 | [S3 Limits, Multipart Upload & Replication](./AWS%20Simple%20Storage%20Service(S3)/S3%20Limits%20,%20Multipart%20upload%20&%20Replication/) | Service quotas, uploading large files, and cross-region replication |
| 5.5 | [Object Ownership and Bucket ACLs](./AWS%20Simple%20Storage%20Service(S3)/Object%20Ownerships%20and%20Bucket%20ACLs/) | Controlling object ownership and access using ACLs |
| 5.6 | [S3 Static Website Hosting](./AWS%20Simple%20Storage%20Service(S3)/S3%20Static%20Website%20hosting/) | Hosting static websites directly from an S3 bucket |
| 5.7 | [Requester Pays & Server Access Logging](./AWS%20Simple%20Storage%20Service(S3)/Requestor%20Pays%20&%20Server%20Access%20Logging/) | Cost attribution for data transfer and auditing bucket access |
| 5.8 | [Lifecycle Configuration](./AWS%20Simple%20Storage%20Service(S3)/Lifecycle%20Configuration/) | Automating object transitions and expirations to reduce costs |
| 5.9 | [Bucket Policy](./AWS%20Simple%20Storage%20Service(S3)/Bucket%20Policy/) | Resource-based policies for controlling access to S3 buckets |
| 5.10 | [S3 Encryption](./AWS%20Simple%20Storage%20Service(S3)/S3%20Encryption/) | SSE-S3, SSE-KMS, SSE-C, and client-side encryption |
| 5.11 | [Transfer Acceleration](./AWS%20Simple%20Storage%20Service(S3)/Transfer%20Acceleration/) | Faster uploads using CloudFront edge locations |
| 5.12 | [S3 ABAC](./AWS%20Simple%20Storage%20Service(S3)/S3%20ABAC/) | Attribute-Based Access Control for dynamic, tag-driven permissions |

---

## 6. AWS Virtual Private Cloud (VPC)

This section provides a detailed understanding of Amazon VPC — how to design, secure, and connect isolated cloud networks on AWS. It covers core networking components and progresses through advanced connectivity and security topics.

📂 **[Explore → AWS Virtual Private Cloud](./Virtual%20Private%20Cloud(VPC)/)**

| # | Sub-Topic | Description |
|---|-----------|-------------|
| 6.1 | [Introduction to AWS VPC](./Virtual%20Private%20Cloud(VPC)/Introduction%20to%20VPC/) | What VPC is, CIDR blocks, and the default VPC |
| 6.2 | [Components of VPC](./Virtual%20Private%20Cloud(VPC)/Components%20of%20VPC/) | Subnets, route tables, internet gateways, NAT gateways, and Elastic IPs |
| 6.3 | [Bastion Host and Private Subnet Connectivity](./Virtual%20Private%20Cloud(VPC)/Bastion%20Hosting/) | Securely accessing private subnet instances using a bastion host |
| 6.4 | [Security Groups vs Network ACLs](./Virtual%20Private%20Cloud(VPC)/NACL%20vs%20Security%20Groups/) | Stateful vs stateless traffic filtering — when to use each |
| 6.5 | [VPC Peering](./Virtual%20Private%20Cloud(VPC)/VPC%20Peering/) | Direct, private connectivity between two VPCs |
| 6.6 | [Virtual Private Network (VPN)](./Virtual%20Private%20Cloud(VPC)/Virtual%20Private%20Network/) | Site-to-Site VPN for connecting on-premises networks to AWS |
| 6.7 | [Transit Gateway](./Virtual%20Private%20Cloud(VPC)/Transit-Gateway/) | Hub-and-spoke routing for connecting multiple VPCs and on-premises networks |
| 6.8 | [VPC Endpoints](./Virtual%20Private%20Cloud(VPC)/VPC-Endpoint/) | Private access to AWS services without internet or NAT — Gateway and Interface endpoints |
| 6.9 | [DNS Firewall](./Virtual%20Private%20Cloud(VPC)/DNS-Firewall/) | Filtering outbound DNS queries to block malicious and unauthorized domains |
| 6.10| [VPC FLOWLOG](./Virtual%20Private%20Cloud(VPC)/VPC-FlowLog/) | Creation and Accessing logs of network traffic to and from the VPC |
| 6.11| [VPC PrivateLink and Lattice](./Virtual%20Private%20Cloud(VPC)/VPC%20PrivateLink%20&%20Lattice/) | Explaining the different methods of private communications - Lattice and PrivateLink|

---
## 7. AWS Elastic Compute Cloud (EC2)

This section covers Amazon EC2 — the backbone of AWS compute. You'll learn how to launch and manage virtual machines, understand instance types and pricing models, configure networking interfaces, control placement and tenancy, and set up web servers on EC2 instances.

📂 **[Explore → AWS Elastic Compute Cloud](./Elastic%20Compute%20Cloud/)**

| # | Sub-Topic | Description |
|---|-----------|-------------|
| 7.1 | [Introduction to EC2](./Elastic%20Compute%20Cloud/Introduction%20to%20EC2/) | What EC2 is, AMIs, instance lifecycle, and launching your first instance |
| 7.2 | [Instance State and Actions](./Elastic%20Compute%20Cloud/Instance%20State%20and%20Actions/) | Describes different Instance States in Lifecycle and Different Actions to execute |
| 7.3 | [Elastic Network Interface](./Elastic%20Compute%20Cloud/Elastic%20Network%20Interface/) | ENIs — attaching, detaching, and managing network interfaces on EC2 |
| 7.4 | [Instance Types and Purchase Options](./Elastic%20Compute%20Cloud/Instance%20Types%20and%20Purchase%20Options/) | On-Demand, Reserved, Spot, and Savings Plans — choosing the right option |
| 7.5 | [Key-Pair and SSH Clients](./Elastic%20Compute%20Cloud/Key-Pair%20and%20SSH%20Clients/) | Generating key pairs, connecting via SSH, and managing access |
| 7.6 | [Placement Groups](./Elastic%20Compute%20Cloud/Placement-Groups/) | Cluster, Spread, and Partition placement strategies for performance and HA |
| 7.7 | [Tenancy and Tenancy Types](./Elastic%20Compute%20Cloud/Tenancy%20and%20Tenancy%20Types/) | Shared, Dedicated Instance, and Dedicated Host — isolation and compliance |
| 7.8 | [Webservers](./Elastic%20Compute%20Cloud/Webservers/) | Installing and configuring web servers (Apache/Nginx) on EC2 instances |
| 7.9 | [Multi-Website in Single Ec2](./Elastic%20Compute%20Cloud/Multi-Server-in-Single-Ec2/) | Installing and configuring Multiple websites in  a single ec2 machine using multiple ips|
| 8.0 | [ssh-keygen](./Elastic%20Compute%20Cloud/ssh-keygen/) | Generating New key-pair incase we lose private key in our machine|
| 8.1 | [LoadBalancers](./Elastic%20Compute%20Cloud/Loadbalancer/) | Introduction to loadbalancers , Loadbalancing Algorithms and Types of Loadbalancers in aws|
| 8.2 | [Network LoadBalancer -Handson](./Elastic%20Compute%20Cloud/NLB/) | Lab demonstrating the Network Loadbalancer setup|
| 8.3 | [Application-LoadBalancer -Handson](./Elastic%20Compute%20Cloud/Application-LoadBalancer/) | Lab demonstrating the Application Loadbalancer setup|
| 8.4 | [Introduction to Autoscaling](./Elastic%20Compute%20Cloud/ASG/) | Introduction to autoscaling and scaling policies|
| 8.5 | [AutoScaling-Hands-on](./Elastic%20Compute%20Cloud/Auto-Scaling-Hands-on/) |Hands-on Lab on autoscaling in aws|
| 8.5 | [Autoscaling+SNS+S3](./Elastic%20Compute%20Cloud/Autoscaling+SNS+S3/) |Hands-on Lab on Autoscaling+SNS+S3|
---
## 8. AWS Elastic Block Store (EBS)

This section provides a detailed understanding of Amazon EBS — how to create, manage, and optimize persistent block storage for EC2 instances. It covers core storage concepts, volume types, performance tuning, snapshots, and security features.

📂 **[Explore → AWS Elastic Block Store (EBS)](./Elastic%20Block%20Store/)**

| # | Sub-Topic | Description |
|---|-----------|-------------|
| 8.1 | [Introduction to AWS EBS](./Elastic%20Block%20Store/Intro%20to%20EBS/) | Block Storage Introduction, EBS and filesystem Types and Hands-on Lab|
| 8.2 | [EBS Permanent Mounting](./Elastic%20Block%20Store/EBS-Permanent-mounting/) | Hands-on Lab on Permanent mouting of ebs volume|
| 8.3 | [Keypair Recovery using EBS](./Elastic%20Block%20Store/Key-Recovery-using-EBS/) | Hands-on Lab on keypair recovery using ebs root volume swapping|
| 8.4 | [Disk-Partition-using-fdisk](./Elastic%20Block%20Store/Disk-Partition-using-fdisk/) | Disk Partitioning using fdisk commandlet|
| 8.5 | [Logical Volume Management](./Elastic%20Block%20Store/LVM/) | logical volume management and hands-on labs |

## 🛠️ Prerequisites

Before diving in, make sure you have:

| Requirement | Details |
|-------------|---------|
| AWS Account | Free tier account is sufficient for most labs |
| AWS CLI | Installed and configured (`aws configure`) |
| Basic Linux | Comfortable with terminal commands |
| IAM User | With appropriate permissions for each service |
| Text Editor | VS Code or similar for editing policy documents |

---

## 🚦 Getting Started

```bash
# Clone this repository
git clone https://github.com/bhuvan-raj/AWS-Zero-to-Hero.git

# Navigate into the project
cd AWS-Zero-to-Hero

# Start with the first section
cd "Introduction to cloud computing"
```

Each folder contains its own `README.md` with step-by-step explanations, diagrams, AWS Console walkthroughs, and CLI commands. Work through the topics in order for the best learning experience.

---

## 🤝 Contributing

Contributions are welcome! If you have suggestions for improvements, new examples, or find any issues, feel free to open an issue or submit a pull request.

---

<div align="center">

**Happy Cloud Building! ☁️**

*If this repo helped you, please consider giving it a ⭐*

</div>
