# Creating a Hosted Zone in AWS Route 53

## 📘 Objective

In this lab, we will learn how to:

- Create a Hosted Zone in AWS Route 53
- Understand Public and Private Hosted Zones
- Configure DNS records
- Understand Name Servers (NS)
- Connect domains with Route 53
- Verify DNS configuration

---

# 📋 Prerequisites

Before starting this lab, ensure you have:

- AWS Account
- IAM user with Route 53 permissions
- Basic understanding of:
  - DNS
  - Domain names
  - IP addresses

---

# 🌐 What is a Hosted Zone?

A Hosted Zone is a container for DNS records in Route 53.

It stores records such as:

- A Record
- CNAME Record
- MX Record
- TXT Record

A Hosted Zone tells Route 53 how to route traffic for a domain.

---

# 🧩 Types of Hosted Zones

## 1. Public Hosted Zone

Used for:

- Public websites
- Internet-facing applications

Accessible from anywhere on the internet.

Example:

```text
example.com
```

---

## 2. Private Hosted Zone

Used for:

- Internal applications
- Internal DNS resolution inside VPCs

Accessible only inside selected VPCs.

Example:

```text
internal.company.local
```

---

# 🏗️ Architecture

## Public Hosted Zone

```text
User
  ↓
Route 53
  ↓
Public Website
```

---

## Private Hosted Zone

```text
EC2 Instance inside VPC
  ↓
Route 53 Private Hosted Zone
  ↓
Internal Application
```

---

# 🚀 Step 1 — Login to AWS Console

1. Open AWS Console
2. Search for:

```text
Route 53
```

3. Open the Route 53 service

---

# 🚀 Step 2 — Open Hosted Zones

From the left sidebar:

```text
Hosted Zones
```

Click:

```text
Create Hosted Zone
```

---

# 🚀 Step 3 — Configure Hosted Zone

## Domain Name

Enter your domain name.

Example:

```text
example.com
```

---

## Type

Choose one:

### Public Hosted Zone

OR

### Private Hosted Zone

---

## Description (Optional)

Example:

```text
Production DNS Zone
```

---

# 🚀 Step 4 — Create Hosted Zone

Click:

```text
Create Hosted Zone
```

Route 53 automatically creates default DNS records.

---

# 📄 Default Records Created

After creation, Route 53 creates:

| Record Type | Purpose |
|---|---|
| NS | Name Servers |
| SOA | Start of Authority |

---

# 🧠 Understanding NS Record

NS (Name Server) records define which DNS servers are authoritative for the domain.

Example:

```text
ns-123.awsdns-45.com
```

These name servers must be configured in your domain registrar.

---

# 🧠 Understanding SOA Record

SOA stands for:

```text
Start of Authority
```

Contains:

- Primary name server
- Admin email
- Serial number
- Refresh settings

---

# 🌍 Public Hosted Zone Workflow

```text
User Browser
   ↓
DNS Resolver
   ↓
Route 53 Name Servers
   ↓
DNS Record
   ↓
Application IP Address
```

---

# 🔒 Private Hosted Zone Workflow

```text
EC2 Instance
   ↓
Route 53 Private Hosted Zone
   ↓
Internal DNS Resolution
```

Only resources inside associated VPCs can access private DNS records.

---

# 📝 Step 5 — Create DNS Records

Inside the Hosted Zone:

Click:

```text
Create Record
```

---

# 🧩 Common DNS Records

## 1. A Record

Maps domain to IPv4 address.

Example:

```text
example.com → 192.168.1.10
```

---

## 2. CNAME Record

Maps one domain to another domain.

Example:

```text
app.example.com → example.com
```

---

## 3. MX Record

Used for email routing.

---

## 4. TXT Record

Used for:

- Domain verification
- SPF
- DKIM

---

# 🚀 Creating an A Record

## Record Name

```text
www
```

---

## Record Type

```text
A – IPv4 Address
```

---

## Value

```text
10.0.0.15
```

OR

Public IP of EC2 instance.

---

## TTL

Example:

```text
300
```

---

# ⏳ Understanding TTL

TTL stands for:

```text
Time To Live
```

Defines how long DNS responses are cached.

Example:

```text
TTL = 300 seconds
```

Lower TTL:
- Faster DNS updates

Higher TTL:
- Better performance

---

# 🌐 Step 6 — Connect Domain Registrar

If you purchased your domain outside AWS:

Example registrars:

- GoDaddy
- Namecheap
- Hostinger

You must update the domain’s Name Servers with Route 53 NS records.

---

# 🔄 Example Name Server Update

Replace old Name Servers with:

```text
ns-123.awsdns-45.com
ns-456.awsdns-78.net
ns-789.awsdns-12.org
ns-321.awsdns-65.co.uk
```

---

# ⏳ DNS Propagation

DNS changes may take time to propagate globally.

Usually:

```text
Few minutes to 48 hours
```

---

# 🔍 Step 7 — Verify DNS

You can verify using:

## Browser

```text
www.example.com
```

---

## Command Line

### Linux/macOS

```bash
nslookup example.com
```

OR

```bash
dig example.com
```

---

### Windows

```powershell
nslookup example.com
```

---

# 🖥️ Example nslookup Output

```text
Server:  8.8.8.8
Address: 8.8.8.8

Name: example.com
Address: 192.168.1.10
```

---

# 🔗 Alias Records

Alias records are AWS-specific DNS records.

Can point directly to:

- Application Load Balancer
- CloudFront
- S3 Static Website
- API Gateway

---

# 🧠 Alias vs CNAME

| Alias | CNAME |
|---|---|
| AWS-specific | Standard DNS |
| Supports root domain | Cannot use root domain |
| No additional DNS lookup | Additional lookup |

---

# 🔒 Private Hosted Zone Configuration

When creating a private hosted zone:

You must associate:

```text
VPC
```

Only resources inside that VPC can resolve the domain.

---

# 🏢 Real-World Use Cases

## Public Hosted Zone

Used for:

- Websites
- APIs
- Public applications

---

## Private Hosted Zone

Used for:

- Internal databases
- Internal microservices
- Corporate applications

---

# 📌 Important Notes

- Hosted Zone stores DNS records
- Public Hosted Zones are internet-facing
- Private Hosted Zones work only inside VPCs
- Route 53 automatically creates NS and SOA records
- Name Servers must be updated in registrar

---

# ⚠️ Common Mistakes

## Incorrect Name Servers

Domain will not resolve properly.

---

## Wrong Record Type

Example:

- Using CNAME instead of A record

---

## High TTL During Migration

DNS updates become slower.

---

# 💡 Best Practices

- Use Alias records for AWS resources
- Use low TTL during migrations
- Separate public and private hosted zones
- Enable health checks for critical applications
- Use descriptive DNS naming

---

# 🧠 Interview Questions

## What is a Hosted Zone?

A container for DNS records in Route 53.

---

## Difference between Public and Private Hosted Zones?

| Public Hosted Zone | Private Hosted Zone |
|---|---|
| Internet accessible | Internal VPC only |
| Used for websites | Used for internal apps |

---

## What records are automatically created?

- NS Record
- SOA Record

---

## What is an NS Record?

Defines authoritative name servers for the domain.

---

## What is TTL?

Defines how long DNS responses are cached.

---

## What happens if Name Servers are incorrect?

DNS resolution fails.

---

# 📌 Important  Tips

Remember:

- Hosted Zone = Container for DNS records
- Public Hosted Zone → Internet-facing
- Private Hosted Zone → VPC-only
- Route 53 automatically creates NS and SOA records
- Alias records are AWS-specific

---
