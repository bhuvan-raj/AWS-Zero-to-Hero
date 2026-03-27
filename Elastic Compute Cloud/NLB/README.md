# AWS Network Load Balancer (NLB) Lab

![AWS](https://img.shields.io/badge/AWS-NLB-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![EC2](https://img.shields.io/badge/EC2-2%20Instances-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Layer](https://img.shields.io/badge/Layer%204-TCP%2FUDP-0080FF?style=for-the-badge)
![Status](https://img.shields.io/badge/Lab-Hands--On-brightgreen?style=for-the-badge)

> A hands-on lab to deploy a Layer 4 Network Load Balancer on AWS, routing traffic across two EC2 instances with health checks and high availability across Availability Zones.

-----

## Architecture

```
User → Internet
         │
         ▼
 ┌───────────────┐
 │  Network Load  │   ← DNS: my-nlb-xxxx.elb.amazonaws.com
 │   Balancer     │
 └──────┬────────┘
        │  TCP :80
        ▼
 ┌─────────────────┐
 │  Target Group   │   ← Health Check: HTTP /
 │ nlb-target-group│
 └────┬───────┬────┘
      │       │
      ▼       ▼
  EC2 AZ-a  EC2 AZ-b
 instance-1 instance-2
```

-----

## Prerequisites

- AWS account with IAM permissions for EC2 and ELB
- SSH key pair (existing or created during lab)
- Basic familiarity with the AWS Console

-----

## Lab Steps

### Step 1 — Launch EC2 Instances

Go to **EC2 → Launch Instance** and create two instances with the following config:

|Setting              |Value                             |
|---------------------|----------------------------------|
|Names                |`nlb-instance-1`, `nlb-instance-2`|
|AMI                  |Amazon Linux 2                    |
|Instance Type        |`t2.micro`                        |
|Network              |Default VPC                       |
|Auto-assign Public IP|Enabled                           |

**Security Group — allow inbound:**

|Port|Protocol|Source   |
|----|--------|---------|
|22  |SSH     |Your IP  |
|80  |HTTP    |0.0.0.0/0|

-----

### Step 2 — Install Web Server

SSH into **each instance** and run:

```bash
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd

echo "Hello from $(hostname)" | sudo tee /var/www/html/index.html
```

-----

### Step 3 — Verify Instances Individually

Open in your browser:

```
http://<instance-1-public-ip>
http://<instance-2-public-ip>
```

✅ Each should display a unique hostname — confirming your web servers are up.

-----

### Step 4 — Create Target Group

Go to **EC2 → Target Groups → Create target group**

|Setting    |Value             |
|-----------|------------------|
|Target type|Instances         |
|Name       |`nlb-target-group`|
|Protocol   |TCP               |
|Port       |80                |
|VPC        |Default           |

**Health Check:**

|Setting |Value|
|--------|-----|
|Protocol|HTTP |
|Path    |`/`  |

Register both EC2 instances → **Include as pending below** → **Create target group**

-----

### Step 5 — Create Network Load Balancer

Go to **EC2 → Load Balancers → Create → Network Load Balancer**

|Setting          |Value          |
|-----------------|---------------|
|Name             |`my-nlb`       |
|Scheme           |Internet-facing|
|IP type          |IPv4           |
|Listener Protocol|TCP            |
|Listener Port    |80             |

Select **at least 2 Availability Zones** with subnets, attach `nlb-target-group`, then click **Create**.

-----

### Step 6 — Wait for Health Checks

Go to **Target Group → Targets tab** and wait until both instances show:

```
Status: healthy
```

> This typically takes 1–2 minutes after NLB provisioning.

-----

### Step 7 — Test the Load Balancer

1. Copy the **DNS name** from your NLB (found in the Load Balancers console)
1. Open in your browser:

```
http://<nlb-dns-name>
```

1. Refresh multiple times — you should see alternating responses:

```
Hello from ip-10-0-1-XXX     ← instance-1
Hello from ip-10-0-2-XXX     ← instance-2
```

✅ Traffic is being distributed across both instances.

-----

