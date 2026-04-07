
# 🚀 Hands-On Lab: Production-Grade Application Hosting on AWS


A comprehensive, step-by-step lab for deploying a highly available, scalable, and secure web application on AWS — done entirely through the **AWS Management Console**.

---

## 📋 Lab Overview

| Field | Details |
|---|---|
| **Difficulty** | Intermediate – Advanced |
| **Estimated Time** | 5 – 7 hours |
| **AWS Region** | `us-east-1` (N. Virginia) — use this throughout |
| **Cost Estimate** | ~$0.50 – $2.00 for the duration of the lab |
| **Approach** | 100% AWS Management Console (GUI) |

### Architecture You'll Build

```
                        ┌──────────────────────────────────────────────────────────────┐
                        │                    VPC (10.0.0.0/16)                         │
                        │                                                              │
Internet ──► IGW ──►    │  ┌──── Public Subnet AZ-A ────┐  ┌── Public Subnet AZ-B ──┐  │
                        │  │    (10.0.1.0/24)           │  │   (10.0.2.0/24)        │  │
                        │  │  ┌──────────────────────┐  │  │ ┌────────────────────┐ │  │
                        │  │  │  ALB (public-facing) │◄─┼──┼─┤   ALB Node (AZ-B)  │ │  │
                        │  │  └──────────┬───────────┘  │  │ └────────────────────┘ │  │
                        │  └────────────┼───────────────┘  └────────────────────────┘  │
                        │               │ (target group)                               │
                        │  ┌─── Private Subnet AZ-A ──┐  ┌─── Private Subnet AZ-B ───┐ │
                        │  │    (10.0.3.0/24)         │  │    (10.0.4.0/24)          │ │
                        │  │  ┌────────────────────┐  │  │  ┌────────────────────┐   │ │
                        │  │  │ EC2 (Auto Scaling) │  │  │  │ EC2 (Auto Scaling) │   │ │
                        │  │  │  IAM Role attached │  │  │  │  IAM Role attached │   │ │
                        │  │  └────────────────────┘  │  │  └────────────────────┘   │ │
                        │  └──────────────────────────┘  └───────────────────────────┘ │
                        │                                                              │
                        │  S3 Bucket (lab-assets-*)  ◄── EC2 via IAM Role              │
                        └──────────────────────────────────────────────────────────────┘
```

---
## ✅ Prerequisites

- An AWS account ([sign up](https://aws.amazon.com/free/))
- A modern browser (Chrome or Firefox recommended)
- Log in to the [AWS Management Console](https://console.aws.amazon.com)
- **Always confirm you are in `us-east-1` (N. Virginia)** — check the region dropdown in the top-right corner before every module

---

## 🔐 Module 1: IAM — Identity and Access Management

> **Navigate to:** Console → Search bar → type `IAM` → click **IAM**

---

### Step 1.1 — Create a Permission Boundary Policy

A permission boundary sets the maximum permissions a role can ever have, even if broader policies are attached later.

1. In the IAM left sidebar, click **Policies** → **Create policy**
2. Click the **JSON** tab and paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket",
        "logs:*",
        "ssm:*",
        "ssmmessages:*",
        "ec2messages:*"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": [
        "iam:*",
        "organizations:*",
        "account:*"
      ],
      "Resource": "*"
    }
  ]
}
```

3. Click **Next** → **Next**
4. Policy name: `lab-ec2-permission-boundary`
5. Click **Create policy**

---

### Step 1.2 — Create the EC2 Instance Policy

1. Click **Policies** → **Create policy** → **JSON** tab and paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ReadStaticAssets",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::lab-assets-*",
        "arn:aws:s3:::lab-assets-*/*"
      ]
    },
    {
      "Sid": "CloudWatchLogs",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Sid": "SSMSessionManager",
      "Effect": "Allow",
      "Action": [
        "ssm:UpdateInstanceInformation",
        "ssmmessages:CreateControlChannel",
        "ssmmessages:CreateDataChannel",
        "ssmmessages:OpenControlChannel",
        "ssmmessages:OpenDataChannel"
      ],
      "Resource": "*"
    }
  ]
}
```

2. Click **Next** → **Next**
3. Policy name: `lab-ec2-instance-policy`
4. Click **Create policy**

---

### Step 1.3 — Create the EC2 IAM Role

1. In the left sidebar, click **Roles** → **Create role**
2. Trusted entity type: **AWS service**
3. Use case: **EC2** → click **Next**
4. In the search box, find and check `lab-ec2-instance-policy` → click **Next**
5. Role name: `lab-ec2-instance-role`
6. Click **Create role**

**Attach the permission boundary:**

7. From the Roles list, click `lab-ec2-instance-role`
8. Click the **Permissions** tab → click **Set permissions boundary**
9. Select **Use a permissions boundary to control the maximum role permissions**
10. Search for and select `lab-ec2-permission-boundary`
11. Click **Set boundary**

> **💡 Key Concept:** The permission boundary doesn't grant permissions — it caps them. The role can only ever do what is in BOTH the attached policy AND the boundary.

---

### Step 1.4 — Create a Lab Deploy User and Group

1. Left sidebar → **User groups** → **Create group**
2. Group name: `lab-deploy-group`
3. Attach these managed policies:
   - `AmazonEC2FullAccess`
   - `AmazonS3FullAccess`
   - `ElasticLoadBalancingFullAccess`
   - `AutoScalingFullAccess`
4. Click **Create group**

**Create the user:**

5. Left sidebar → **Users** → **Create user**
6. Username: `lab-deploy-user`
7. Check **Provide user access to the AWS Management Console** → set a password
8. Click **Next** → select **Add user to group** → select `lab-deploy-group`
9. Click **Next** → **Create user**
10. Download the credentials CSV and store it safely

---

## 🌐 Module 2: VPC — Virtual Private Cloud

> **Navigate to:** Console → Search bar → `VPC` → click **VPC**

---

### Step 2.1 — Create the VPC

1. Left sidebar → **Your VPCs** → **Create VPC**
2. Select **VPC only**
3. Fill in:
   | Field | Value |
   |---|---|
   | Name tag | `lab-vpc` |
   | IPv4 CIDR | `10.0.0.0/16` |
   | Tenancy | Default |
4. Click **Create VPC**

**Enable DNS settings:**

5. Select `lab-vpc` → click **Actions** → **Edit VPC settings**
6. Check both:
   - ✅ Enable DNS hostnames
   - ✅ Enable DNS resolution
7. Click **Save**

---

### Step 2.2 — Create Four Subnets

Go to **Subnets** → **Create subnet** → select `lab-vpc`. Create all four:

**Public Subnet — AZ A**
| Field | Value |
|---|---|
| Subnet name | `lab-public-subnet-a` |
| Availability Zone | `us-east-1a` |
| IPv4 CIDR | `10.0.1.0/24` |

**Public Subnet — AZ B**
| Field | Value |
|---|---|
| Subnet name | `lab-public-subnet-b` |
| Availability Zone | `us-east-1b` |
| IPv4 CIDR | `10.0.2.0/24` |

**Private Subnet — AZ A**
| Field | Value |
|---|---|
| Subnet name | `lab-private-subnet-a` |
| Availability Zone | `us-east-1a` |
| IPv4 CIDR | `10.0.3.0/24` |

**Private Subnet — AZ B**
| Field | Value |
|---|---|
| Subnet name | `lab-private-subnet-b` |
| Availability Zone | `us-east-1b` |
| IPv4 CIDR | `10.0.4.0/24` |

**Enable auto-assign public IP on both public subnets:**

For each of `lab-public-subnet-a` and `lab-public-subnet-b`:
1. Select the subnet → **Actions** → **Edit subnet settings**
2. Check ✅ **Enable auto-assign public IPv4 address**
3. Click **Save**

---

### Step 2.3 — Create and Attach an Internet Gateway

1. Left sidebar → **Internet gateways** → **Create internet gateway**
2. Name: `lab-igw` → **Create internet gateway**
3. Select `lab-igw` → **Actions** → **Attach to VPC** → select `lab-vpc` → **Attach**

---

### Step 2.4 — Create NAT Gateways (One Per AZ)

NAT Gateways let private subnet instances reach the internet for updates without being publicly accessible.

**NAT Gateway for AZ-A:**

1. Left sidebar → **NAT gateways** → **Create NAT gateway**
2. Fill in:
   | Field | Value |
   |---|---|
   | Name | `lab-nat-a` |
   | Subnet | `lab-public-subnet-a` |
   | Connectivity type | Public |
3. Click **Allocate Elastic IP** → then **Create NAT gateway**

**NAT Gateway for AZ-B:**

4. Click **Create NAT gateway** again
5. Fill in:
   | Field | Value |
   |---|---|
   | Name | `lab-nat-b` |
   | Subnet | `lab-public-subnet-b` |
   | Connectivity type | Public |
6. Click **Allocate Elastic IP** → then **Create NAT gateway**

⏳ Wait for both NAT Gateways to show **Available** status before continuing (~2 minutes).

---

### Step 2.5 — Create Route Tables

**Public Route Table:**

1. Left sidebar → **Route tables** → **Create route table**
2. Name: `lab-public-rtb` | VPC: `lab-vpc` → **Create**
3. Select `lab-public-rtb` → **Routes** tab → **Edit routes** → **Add route**:
   - Destination: `0.0.0.0/0` | Target: `lab-igw`
4. Click **Save changes**
5. **Subnet associations** tab → **Edit subnet associations** → select both public subnets → **Save**

**Private Route Table — AZ-A:**

6. Create route table: name `lab-private-rtb-a` | VPC: `lab-vpc`
7. **Routes** → **Edit routes** → **Add route**:
   - Destination: `0.0.0.0/0` | Target: `lab-nat-a`
8. **Subnet associations** → select `lab-private-subnet-a` → **Save**

**Private Route Table — AZ-B:**

9. Create route table: name `lab-private-rtb-b` | VPC: `lab-vpc`
10. **Routes** → **Edit routes** → **Add route**:
    - Destination: `0.0.0.0/0` | Target: `lab-nat-b`
11. **Subnet associations** → select `lab-private-subnet-b` → **Save**

---

### Step 2.6 — Create Security Groups

> **Navigate to:** VPC → **Security groups**

**ALB Security Group:**

1. **Create security group**
   | Field | Value |
   |---|---|
   | Name | `lab-alb-sg` |
   | Description | Allow HTTP and HTTPS from internet |
   | VPC | `lab-vpc` |
2. **Inbound rules** → **Add rule** twice:
   | Type | Protocol | Port | Source |
   |---|---|---|---|
   | HTTP | TCP | 80 | `0.0.0.0/0` |
   | HTTPS | TCP | 443 | `0.0.0.0/0` |
3. Click **Create security group**

**EC2 Security Group:**

4. **Create security group**
   | Field | Value |
   |---|---|
   | Name | `lab-ec2-sg` |
   | Description | Allow HTTP from ALB only |
   | VPC | `lab-vpc` |
5. **Inbound rules** → **Add rule**:
   | Type | Protocol | Port | Source |
   |---|---|---|---|
   | HTTP | TCP | 80 | Custom → select `lab-alb-sg` |
6. Click **Create security group**

> **🔒 Security Note:** By using the ALB security group as the source (not a CIDR block), you ensure EC2 instances only accept traffic that has passed through the ALB — even if someone discovers a private IP, they can't reach the instance directly.

---

## 🪣 Module 3: S3 — Static Assets Bucket

> **Navigate to:** Console → Search bar → `S3` → click **S3**

---

### Step 3.1 — Create the Bucket

1. Click **Create bucket**
2. Fill in:
   | Field | Value |
   |---|---|
   | Bucket name | `lab-assets-YOUR-ACCOUNT-ID` (replace with your 12-digit account ID) |
   | AWS Region | `us-east-1` |
3. Under **Block Public Access settings** — leave all four checkboxes ✅ checked (default)
4. Click **Create bucket**

> **📌 Tip:** Find your Account ID in the top-right of the console (click your username).

---

### Step 3.2 — Enable Versioning

1. Open the bucket → **Properties** tab
2. Scroll to **Bucket Versioning** → click **Edit**
3. Select **Enable** → **Save changes**

---

### Step 3.3 — Enable Server-Side Encryption

1. **Properties** tab → scroll to **Default encryption** → **Edit**
2. Encryption type: **Server-side encryption with Amazon S3 managed keys (SSE-S3)**
3. Bucket key: **Enable**
4. Click **Save changes**

---

### Step 3.4 — Create a Lifecycle Rule

1. **Management** tab → **Create lifecycle rule**
2. Fill in:
   | Field | Value |
   |---|---|
   | Rule name | `transition-old-versions` |
   | Rule scope | Apply to all objects |
3. Check ✅ **Transition noncurrent versions of objects between storage classes**
   - Storage class: **Standard-IA**
   - Days after objects become noncurrent: `30`
4. Check ✅ **Permanently delete noncurrent versions of objects**
   - Days after objects become noncurrent: `90`
5. Check ✅ **Delete expired object delete markers or incomplete multipart uploads**
   - Select **Delete incomplete multipart uploads** → Days: `7`
6. Click **Create rule**

---

### Step 3.5 — Apply a Bucket Policy

1. **Permissions** tab → scroll to **Bucket policy** → **Edit**
2. Paste the policy below, replacing `YOUR-ACCOUNT-ID` and `YOUR-BUCKET-NAME`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2RoleAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::YOUR-ACCOUNT-ID:role/lab-ec2-instance-role"
      },
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::YOUR-BUCKET-NAME",
        "arn:aws:s3:::YOUR-BUCKET-NAME/*"
      ]
    },
    {
      "Sid": "DenyAllOtherAccess",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::YOUR-BUCKET-NAME",
        "arn:aws:s3:::YOUR-BUCKET-NAME/*"
      ],
      "Condition": {
        "StringNotLike": {
          "aws:PrincipalArn": [
            "arn:aws:iam::YOUR-ACCOUNT-ID:role/lab-ec2-instance-role",
            "arn:aws:iam::YOUR-ACCOUNT-ID:root"
          ]
        }
      }
    }
  ]
}
```

3. Click **Save changes**

> **💡 Key Concept:** The `Deny` statement with `StringNotLike` creates an explicit deny for every principal EXCEPT the EC2 role and root. In IAM, explicit denies always override allows.

---

### Step 3.6 — Upload Sample Assets

1. In your bucket, click **Upload** → **Add files**
2. Create a file named `styles.css` on your computer with this content, then upload it:

```css
body {
  font-family: Arial, sans-serif;
  background: #f4f4f4;
  text-align: center;
  padding: 40px;
}
h1 { color: #FF9900; }
p { color: #333; }
```

3. After selecting the file, expand **Additional upload options**
   - Content type: `text/css`
   - Cache control: `max-age=86400`
4. Click **Upload**
5. Once uploaded, move it into a folder: select the file → **Actions** → **Move** → destination: `assets/styles.css` inside the same bucket

---

## ⚖️ Module 4: Application Load Balancer

> **Navigate to:** Console → `EC2` → left sidebar → **Load Balancers**

---

### Step 4.1 — Create the Target Group First

1. Left sidebar → **Target Groups** → **Create target group**
2. Fill in:
   | Field | Value |
   |---|---|
   | Target type | Instances |
   | Target group name | `lab-target-group` |
   | Protocol | HTTP |
   | Port | 80 |
   | VPC | `lab-vpc` |
3. Under **Health checks**:
   | Field | Value |
   |---|---|
   | Health check path | `/health` |
   | Healthy threshold | 2 |
   | Unhealthy threshold | 3 |
   | Timeout | 5 seconds |
   | Interval | 30 seconds |
   | Success codes | 200 |
4. Click **Next** → skip registering targets for now → **Create target group**

---

### Step 4.2 — Create the Application Load Balancer

1. Left sidebar → **Load Balancers** → **Create load balancer**
2. Select **Application Load Balancer** → **Create**
3. Fill in the **Basic configuration**:
   | Field | Value |
   |---|---|
   | Name | `lab-alb` |
   | Scheme | Internet-facing |
   | IP address type | IPv4 |

4. **Network mapping:**
   - VPC: `lab-vpc`
   - Mappings: check both `us-east-1a` → select `lab-public-subnet-a` and `us-east-1b` → select `lab-public-subnet-b`

5. **Security groups:** remove the default, select `lab-alb-sg`

6. **Listeners and routing:**
   - Protocol: HTTP | Port: 80
   - Default action: Forward to → `lab-target-group`

7. Click **Create load balancer**

8. Once created, note the **DNS name** (e.g., `lab-alb-123456.us-east-1.elb.amazonaws.com`) — you'll use this for testing.

---

### Step 4.3 — Add a Custom Listener Rule for /health

1. Click on `lab-alb` → **Listeners** tab → click on the **HTTP:80** listener
2. Click **Add rule** → **Add rule**
3. Name: `health-check-rule`
4. Click **Add condition** → **Path** → `/health` → confirm
5. Click **Add action** → **Return fixed response**
   - Response code: `200`
   - Content-Type: `text/plain`
   - Response body: `OK`
6. Priority: `1` → **Create**

---

## 📦 Module 5: EC2 Launch Template

> **Navigate to:** Console → `EC2` → left sidebar → **Launch Templates**

---

### Step 5.1 — Create a Key Pair (Optional)

> Skip this if you plan to use SSM Session Manager only (recommended).

1. Left sidebar → **Key Pairs** → **Create key pair**
2. Name: `lab-key-pair` | Format: `.pem` (Mac/Linux) or `.ppk` (Windows)
3. Click **Create key pair** — the file downloads automatically. Store it safely.

---

### Step 5.2 — Create the Launch Template

1. Left sidebar → **Launch Templates** → **Create launch template**

**Template settings:**
| Field | Value |
|---|---|
| Launch template name | `lab-launch-template` |
| Template version description | `v1-initial` |

**Application and OS Images:**
1. Click **Browse more AMIs** → search `Amazon Linux 2023`
2. Select the latest **Amazon Linux 2023 AMI** (64-bit x86) → **Select**

**Instance type:** `t3.micro`

**Key pair:** Select `lab-key-pair` (or **Don't include in launch template** if using SSM)

**Network settings:**
- Don't include a subnet (ASG will choose)
- Security groups: select `lab-ec2-sg`

**Advanced details — IAM instance profile:**
- IAM instance profile: `lab-ec2-profile`

  > If you don't see this, go back to IAM → Instance Profiles and confirm the profile was created in Step 1.3.

**Advanced details — Metadata (IMDSv2):**
- Metadata accessible: **Enabled**
- Metadata version: **V2 only (token required)**
- Allowed hop limit: `1`

**Advanced details — Detailed CloudWatch monitoring:** **Enable**

**Advanced details — User data:**

Paste the following script exactly:

```bash
#!/bin/bash
set -e
yum update -y
yum install -y httpd

systemctl start httpd
systemctl enable httpd

# Health check endpoint
echo "OK" > /var/www/html/health

# Get instance metadata
INSTANCE_ID=$(TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600") && \
  curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id)

AZ=$(TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600") && \
  curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/placement/availability-zone)

ACCOUNT_ID=$(TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600") && \
  curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/dynamic/instance-identity/document | \
  python3 -c 'import sys,json; print(json.load(sys.stdin)["accountId"])')

BUCKET_NAME="lab-assets-${ACCOUNT_ID}"

# Pull static assets from S3
mkdir -p /var/www/html/assets
aws s3 cp s3://${BUCKET_NAME}/assets/styles.css /var/www/html/assets/styles.css \
  --region us-east-1 || true

# Write the app page
cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
  <title>AWS Lab App</title>
  <link rel="stylesheet" href="/assets/styles.css">
</head>
<body>
  <h1>🚀 My AWS Lab Application</h1>
  <p><strong>Instance ID:</strong> ${INSTANCE_ID}</p>
  <p><strong>Availability Zone:</strong> ${AZ}</p>
  <p>Routed via ALB → Auto Scaling Group → EC2 (private subnet)</p>
</body>
</html>
EOF
```

Click **Create launch template**

---

## 📈 Module 6: Auto Scaling Group

> **Navigate to:** Console → `EC2` → left sidebar → **Auto Scaling Groups**

---

### Step 6.1 — Create the Auto Scaling Group

1. Click **Create Auto Scaling group**

**Step 1 — Choose launch template:**
| Field | Value |
|---|---|
| Auto Scaling group name | `lab-asg` |
| Launch template | `lab-launch-template` |
| Version | Latest |

Click **Next**

**Step 2 — Choose instance launch options:**
- VPC: `lab-vpc`
- Availability Zones and subnets: select both `lab-private-subnet-a` and `lab-private-subnet-b`

Click **Next**

**Step 3 — Configure advanced options:**
- Load balancing: **Attach to an existing load balancer**
- Choose from your load balancer target groups: `lab-target-group`
- Health checks:
  - ✅ Turn on Elastic Load Balancing health checks
  - Health check grace period: `120` seconds
- Monitoring: ✅ Enable group metrics collection within CloudWatch

Click **Next**

**Step 4 — Configure group size and scaling:**
| Field | Value |
|---|---|
| Desired capacity | `2` |
| Minimum capacity | `2` |
| Maximum capacity | `6` |

**Automatic scaling — Add a scaling policy:**
- Select **Target tracking scaling policy**

First policy:
| Field | Value |
|---|---|
| Scaling policy name | `lab-cpu-tracking` |
| Metric type | Average CPU utilization |
| Target value | `60` |
| Instance warmup | `120` seconds |

Click **Add policy** again for a second policy:
| Field | Value |
|---|---|
| Scaling policy name | `lab-request-tracking` |
| Metric type | Application Load Balancer request count per target |
| Target group | `lab-target-group` |
| Target value | `1000` |
| Instance warmup | `120` seconds |

Click **Next**

**Step 5 — Add notifications:** Skip → **Next**

**Step 6 — Add tags:**
| Key | Value | Propagate to instances |
|---|---|---|
| `Name` | `lab-web-server` | ✅ |
| `Environment` | `lab` | ✅ |

Click **Next** → **Create Auto Scaling group**

---

### Step 6.2 — Add Scheduled Scaling Actions

1. Open `lab-asg` → **Automatic scaling** tab → scroll to **Scheduled actions**
2. Click **Create scheduled action**

**Scale up (business hours):**
| Field | Value |
|---|---|
| Name | `scale-up-business-hours` |
| Desired capacity | `4` |
| Min | `4` |
| Max | `6` |
| Recurrence | `0 8 * * MON-FRI` (cron — 8am UTC weekdays) |

3. Click **Create** → then **Create scheduled action** again

**Scale down (off-hours):**
| Field | Value |
|---|---|
| Name | `scale-down-off-hours` |
| Desired capacity | `2` |
| Min | `2` |
| Max | `6` |
| Recurrence | `0 20 * * MON-FRI` (cron — 8pm UTC weekdays) |

4. Click **Create**

---

### Step 6.3 — Test Instance Refresh (Rolling Update)

Instance Refresh lets you roll out changes to all instances with zero downtime.

1. Open `lab-asg` → **Instance refresh** tab
2. Click **Start instance refresh**
3. Configure:
   | Field | Value |
   |---|---|
   | Minimum healthy percentage | `80` |
   | Instance warmup | `120` seconds |
   | Enable checkpoints | ✅ Yes |
   | Checkpoint percentages | `33`, `66`, `100` |
   | Checkpoint delay | `120` seconds |
4. Click **Start instance refresh**
5. Watch the **Status** column — it will move through `Pending → InProgress → Successful`

---

## ✅ Module 7: Validation and Testing

### Step 7.1 — Check Target Group Health

1. Go to **EC2** → **Target Groups** → click `lab-target-group`
2. Click the **Targets** tab
3. Wait until all registered instances show **healthy** status (green)

If any instance shows **unhealthy**, click on it → check **Health status details** for the reason (usually the instance is still warming up — wait 2–3 minutes).

---

### Step 7.2 — Test the Application in a Browser

1. Go to **EC2** → **Load Balancers** → click `lab-alb`
2. Copy the **DNS name** from the Description tab
3. Open a browser and visit:
   - `http://<ALB-DNS-NAME>/` — should show the app page with Instance ID and AZ
   - `http://<ALB-DNS-NAME>/health` — should return `OK`
   - `http://<ALB-DNS-NAME>/assets/styles.css` — should return your CSS file

4. **Refresh the page 5–10 times.** The Instance ID should alternate between two different IDs — confirming the ALB is distributing traffic across both instances in different AZs.

---

### Step 7.3 — Connect to an Instance Without SSH (SSM Session Manager)

1. Go to **EC2** → **Instances**
2. Select one of the running `lab-web-server` instances
3. Click **Connect** (top right)
4. Select the **Session Manager** tab → click **Connect**

A browser-based terminal opens — no SSH, no key pair, no open ports needed.

Inside the session:
```bash
# Verify the web server is running
systemctl status httpd

# Confirm S3 assets were pulled
ls /var/www/html/assets/

# Check the health endpoint file
cat /var/www/html/health
```

---

### Step 7.4 — Simulate Load and Watch Auto Scaling

Inside the SSM session on an EC2 instance:

```bash
# Install stress tool and generate CPU load
sudo yum install -y stress
sudo stress --cpu 4 --timeout 300
```

Back in the console, watch the ASG respond:

1. Go to **EC2** → **Auto Scaling Groups** → `lab-asg`
2. Click the **Activity** tab — you'll see scale-out events appear within 2–3 minutes
3. Click the **Instance management** tab — watch new instances appear and register with the target group
4. After the stress test ends, watch the ASG scale back in after the cooldown period (~5 minutes)

---

## 🧪 Lab Validation Checklist

**Module 1 – IAM**
- [ ] Permission boundary policy `lab-ec2-permission-boundary` created
- [ ] Instance policy `lab-ec2-instance-policy` created (S3, CloudWatch, SSM)
- [ ] Role `lab-ec2-instance-role` created, policy attached, boundary set
- [ ] Deploy user `lab-deploy-user` added to `lab-deploy-group`

**Module 2 – VPC**
- [ ] VPC with CIDR `10.0.0.0/16`, DNS hostnames enabled
- [ ] 2 public + 2 private subnets across AZ-A and AZ-B
- [ ] Public subnets have auto-assign public IP enabled
- [ ] Internet Gateway attached to VPC
- [ ] 2 NAT Gateways (one per AZ) with Elastic IPs, both showing **Available**
- [ ] Public route table: `0.0.0.0/0` → IGW, associated with both public subnets
- [ ] Private route tables AZ-A and AZ-B: `0.0.0.0/0` → respective NAT Gateway
- [ ] `lab-alb-sg`: allows ports 80 and 443 from `0.0.0.0/0`
- [ ] `lab-ec2-sg`: allows port 80 from `lab-alb-sg` only

**Module 3 – S3**
- [ ] Bucket created with all public access blocked
- [ ] Versioning enabled
- [ ] SSE-S3 encryption with Bucket Key enabled
- [ ] Lifecycle rule created (30-day IA transition, 90-day expiry)
- [ ] Bucket policy saved (allows EC2 role, denies all others)
- [ ] `styles.css` uploaded to `assets/styles.css`

**Module 4 – ALB**
- [ ] Target group `lab-target-group` with `/health` path, HTTP 200 matcher
- [ ] ALB `lab-alb` deployed across both public subnets (internet-facing)
- [ ] ALB uses `lab-alb-sg`, listener on port 80 forwards to target group
- [ ] Custom listener rule: path `/health` → fixed 200 response

**Module 5 – Launch Template**
- [ ] Latest Amazon Linux 2023 AMI selected
- [ ] Instance type `t3.micro`
- [ ] Security group `lab-ec2-sg` attached
- [ ] IAM instance profile `lab-ec2-profile` set
- [ ] IMDSv2 enforced (V2 only, hop limit 1)
- [ ] Detailed monitoring enabled
- [ ] User data script pasted correctly

**Module 6 – Auto Scaling**
- [ ] ASG spans `lab-private-subnet-a` and `lab-private-subnet-b`
- [ ] Min=2, Max=6, Desired=2
- [ ] ALB target group attached, ELB health checks on, 120s grace period
- [ ] CPU target tracking at 60%
- [ ] Request count tracking at 1000 req/target
- [ ] Scheduled scale-up (8am Mon–Fri) and scale-down (8pm Mon–Fri) created
- [ ] Instance Refresh ran successfully

**Module 7 – Validation**
- [ ] All 2 targets healthy in target group
- [ ] `http://<ALB-DNS>/` returns the styled HTML page
- [ ] `http://<ALB-DNS>/health` returns `OK`
- [ ] Refreshing the page shows different Instance IDs (load balancing works)
- [ ] `http://<ALB-DNS>/assets/styles.css` returns the CSS file from S3
- [ ] SSM Session Manager connected to a private instance successfully
- [ ] ASG scaled out during CPU stress test; scaled in after cooldown

---

## 🧹 Cleanup — Avoid Unexpected Charges

> ⚠️ **Follow this order exactly.** Deleting in the wrong order causes dependency errors.

| Step | Service | Action |
|---|---|---|
| 1 | Auto Scaling | Open `lab-asg` → **Actions** → **Delete** → confirm |
| 2 | Launch Template | Select `lab-launch-template` → **Actions** → **Delete** |
| 3 | ALB | Select `lab-alb` → **Actions** → **Delete** |
| 4 | Target Group | Select `lab-target-group` → **Actions** → **Delete** |
| 5 | S3 | Open bucket → **Empty** (confirm) → then **Delete bucket** (confirm) |
| 6 | NAT Gateways | Select `lab-nat-a` → **Actions** → **Delete** → repeat for `lab-nat-b` → wait for **Deleted** status |
| 7 | Elastic IPs | Go to **Elastic IPs** → release both unassociated IPs |
| 8 | Security Groups | Delete `lab-ec2-sg`, then `lab-alb-sg` |
| 9 | Subnets | Delete all 4 subnets |
| 10 | Route Tables | Delete `lab-public-rtb`, `lab-private-rtb-a`, `lab-private-rtb-b` |
| 11 | Internet Gateway | Detach from VPC → then Delete |
| 12 | VPC | Select `lab-vpc` → **Actions** → **Delete** |
| 13 | IAM | Delete role `lab-ec2-instance-role` → delete instance profile `lab-ec2-profile` → delete both policies → delete user and group |
| 14 | EC2 Key Pair | Delete `lab-key-pair` from Key Pairs console page |

---

## 🔗 Further Reading

| Topic | Link |
|---|---|
| IAM Best Practices | https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html |
| IAM Permission Boundaries | https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html |
| VPC with Private Subnets & NAT | https://docs.aws.amazon.com/vpc/latest/userguide/vpc-example-private-subnets-nat.html |
| ALB Listener Rules | https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-update-rules.html |
| Auto Scaling Target Tracking | https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-target-tracking.html |
| Auto Scaling Instance Refresh | https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-instance-refresh.html |
| S3 Bucket Policies | https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html |
| S3 Lifecycle Rules | https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html |
| IMDSv2 | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html |
| SSM Session Manager | https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html |

---

## 📝 Lab Summary

| Module | Service | Key Concepts |
|---|---|---|
| 1 | IAM | Policies, permission boundaries, roles, instance profiles, users & groups |
| 2 | VPC | Multi-AZ subnets, IGW, NAT Gateways, route tables, security group chaining |
| 3 | S3 | Bucket policies, versioning, lifecycle rules, SSE encryption |
| 4 | ALB | Target groups, health checks, listener rules, fixed-response actions |
| 5 | EC2 | Launch Templates, IMDSv2, user data bootstrapping |
| 6 | Auto Scaling | Target tracking, scheduled scaling, instance refresh |
| 7 | Validation | SSM Session Manager, load testing, ASG scaling observation |

---

*Happy building on AWS! ☁️*
