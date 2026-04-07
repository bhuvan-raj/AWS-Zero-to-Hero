# 🚀 Hands-On Lab: Production-Grade Application Hosting on AWS

A comprehensive, end-to-end lab for deploying a highly available, scalable, and secure web application on AWS using industry best practices.

---

## 📋 Lab Overview

| Field | Details |
|---|---|
| **Difficulty** | Intermediate – Advanced |
| **Estimated Time** | 5 – 7 hours |
| **AWS Region** | `us-east-1` (N. Virginia) |
| **Cost Estimate** | ~$0.50 – $2.00 for the duration of the lab |
| **Architecture** | Multi-AZ, Auto Scaling, Load Balanced |

### What You'll Build

A production-like, highly available web application stack with:

- A hardened **VPC** spanning two Availability Zones with public and private subnets
- **IAM** roles, policies, permission boundaries, and an instance profile
- An **Application Load Balancer** terminating HTTP traffic
- An **Auto Scaling Group** launching EC2 instances in private subnets
- An **S3 bucket** with bucket policies, versioning, and lifecycle rules serving static assets
- **Security Groups** at every layer enforcing least-privilege network access

```
                        ┌─────────────────────────────────────────────────────────┐
                        │                    VPC (10.0.0.0/16)                    │
                        │                                                         │
Internet ──► IGW ──►   │  ┌──── Public Subnet AZ-A ────┐  ┌── Public Subnet AZ-B ──┐  │
                        │  │    (10.0.1.0/24)            │  │   (10.0.2.0/24)        │  │
                        │  │  ┌──────────────────────┐   │  │ ┌────────────────────┐ │  │
                        │  │  │  ALB (public-facing)  │◄──┼──┼─┤   ALB Node (AZ-B)  │ │  │
                        │  │  └──────────┬───────────┘   │  │ └────────────────────┘ │  │
                        │  └────────────┼────────────────┘  └────────────────────────┘  │
                        │               │ (target group)                                 │
                        │  ┌─── Private Subnet AZ-A ──┐  ┌─── Private Subnet AZ-B ───┐  │
                        │  │    (10.0.3.0/24)          │  │    (10.0.4.0/24)           │  │
                        │  │  ┌────────────────────┐   │  │  ┌────────────────────┐   │  │
                        │  │  │ EC2 (Auto Scaling)  │   │  │  │ EC2 (Auto Scaling)  │   │  │
                        │  │  │  IAM Role attached  │   │  │  │  IAM Role attached  │   │  │
                        │  │  └────────────────────┘   │  │  └────────────────────┘   │  │
                        │  └───────────────────────────┘  └────────────────────────────┘  │
                        │                                                         │
                        │  S3 Bucket (lab-assets-*)  ◄── EC2 via IAM Role        │
                        └─────────────────────────────────────────────────────────┘
```

---

## 🎯 Learning Objectives

By the end of this lab you will be able to:

- Design and implement a multi-AZ VPC with public and private subnet tiers
- Author IAM policies using least-privilege principles, attach permission boundaries, and use instance profiles
- Write and attach S3 bucket policies, enable versioning, and configure lifecycle rules
- Deploy and configure an Application Load Balancer with listener rules and health checks
- Create a Launch Template and configure an Auto Scaling Group with scaling policies
- Understand how all these services integrate into a resilient production architecture

---

## ✅ Prerequisites

- An AWS account with administrator access ([sign up here](https://aws.amazon.com/free/))
- AWS CLI v2 installed and configured (`aws configure`)
- Basic familiarity with Linux CLI and JSON
- An SSH client (Terminal on macOS/Linux, Windows Terminal on Windows)

```bash
# Verify your CLI is set up correctly
aws sts get-caller-identity
```

---

## 🔐 Module 1: IAM – Identity and Access Management

**Goal:** Create a least-privilege IAM role for EC2 instances, a deployment user, and a permission boundary.

### Step 1.1 – Create the EC2 Instance Role Policy Document

This policy grants EC2 instances read access to S3 and the ability to write CloudWatch logs.

Create a file named `ec2-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ReadStaticAssets",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
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
        "logs:PutLogEvents",
        "logs:DescribeLogStreams"
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

```bash
# Create the IAM policy
aws iam create-policy \
  --policy-name lab-ec2-instance-policy \
  --policy-document file://ec2-policy.json
```

### Step 1.2 – Create a Permission Boundary

A permission boundary caps the maximum permissions any role can have. Even if a misconfiguration grants broader access, the boundary enforces the ceiling.

Create `permission-boundary.json`:

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

```bash
aws iam create-policy \
  --policy-name lab-ec2-permission-boundary \
  --policy-document file://permission-boundary.json
```

### Step 1.3 – Create the EC2 IAM Role with Trust Policy

Create `ec2-trust-policy.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

```bash
# Get your AWS account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Create the role with the permission boundary
aws iam create-role \
  --role-name lab-ec2-instance-role \
  --assume-role-policy-document file://ec2-trust-policy.json \
  --permissions-boundary arn:aws:iam::${ACCOUNT_ID}:policy/lab-ec2-permission-boundary

# Attach the instance policy to the role
aws iam attach-role-policy \
  --role-name lab-ec2-instance-role \
  --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/lab-ec2-instance-policy

# Create an instance profile and add the role to it
aws iam create-instance-profile --instance-profile-name lab-ec2-profile
aws iam add-role-to-instance-profile \
  --instance-profile-name lab-ec2-profile \
  --role-name lab-ec2-instance-role
```

### Step 1.4 – Create a Lab Deployment IAM User

```bash
# Create user
aws iam create-user --user-name lab-deploy-user

# Create a group with scoped deploy permissions
aws iam create-group --group-name lab-deploy-group

# Attach policies to the group
aws iam attach-group-policy \
  --group-name lab-deploy-group \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess

aws iam attach-group-policy \
  --group-name lab-deploy-group \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

aws iam attach-group-policy \
  --group-name lab-deploy-group \
  --policy-arn arn:aws:iam::aws:policy/ElasticLoadBalancingFullAccess

aws iam attach-group-policy \
  --group-name lab-deploy-group \
  --policy-arn arn:aws:iam::aws:policy/AutoScalingFullAccess

# Add user to group
aws iam add-user-to-group \
  --user-name lab-deploy-user \
  --group-name lab-deploy-group

# Generate access keys for CLI use
aws iam create-access-key --user-name lab-deploy-user
```

> **💡 Best Practice:** Save the `AccessKeyId` and `SecretAccessKey` output securely. You cannot retrieve the secret key again. In real environments, prefer IAM Identity Center (SSO) over long-term access keys.

---

## 🌐 Module 2: VPC – Virtual Private Cloud

**Goal:** Build a multi-AZ VPC with public and private subnets, NAT Gateways, and a strict security group hierarchy.

### Step 2.1 – Create the VPC

```bash
# Create the VPC
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --query 'Vpc.VpcId' \
  --output text)

# Enable DNS hostnames (required for ALB and SSM)
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-support

# Tag it
aws ec2 create-tags --resources $VPC_ID --tags Key=Name,Value=lab-vpc

echo "VPC ID: $VPC_ID"
```

### Step 2.2 – Create Subnets (2 Public, 2 Private)

```bash
# Public Subnet – AZ A
PUB_SUB_A=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --query 'Subnet.SubnetId' --output text)
aws ec2 create-tags --resources $PUB_SUB_A --tags Key=Name,Value=lab-public-subnet-a

# Public Subnet – AZ B
PUB_SUB_B=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1b \
  --query 'Subnet.SubnetId' --output text)
aws ec2 create-tags --resources $PUB_SUB_B --tags Key=Name,Value=lab-public-subnet-b

# Private Subnet – AZ A
PRIV_SUB_A=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.3.0/24 \
  --availability-zone us-east-1a \
  --query 'Subnet.SubnetId' --output text)
aws ec2 create-tags --resources $PRIV_SUB_A --tags Key=Name,Value=lab-private-subnet-a

# Private Subnet – AZ B
PRIV_SUB_B=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.4.0/24 \
  --availability-zone us-east-1b \
  --query 'Subnet.SubnetId' --output text)
aws ec2 create-tags --resources $PRIV_SUB_B --tags Key=Name,Value=lab-private-subnet-b

# Enable auto-assign public IP on public subnets only
aws ec2 modify-subnet-attribute --subnet-id $PUB_SUB_A --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id $PUB_SUB_B --map-public-ip-on-launch
```

### Step 2.3 – Internet Gateway

```bash
IGW_ID=$(aws ec2 create-internet-gateway \
  --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
aws ec2 create-tags --resources $IGW_ID --tags Key=Name,Value=lab-igw
```

### Step 2.4 – NAT Gateways (One Per AZ for High Availability)

NAT Gateways allow instances in private subnets to reach the internet (e.g., for package updates) without being publicly reachable.

```bash
# Allocate Elastic IPs for NAT Gateways
EIP_A=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
EIP_B=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)

# Create NAT Gateways in the PUBLIC subnets
NAT_A=$(aws ec2 create-nat-gateway \
  --subnet-id $PUB_SUB_A \
  --allocation-id $EIP_A \
  --query 'NatGateway.NatGatewayId' --output text)
aws ec2 create-tags --resources $NAT_A --tags Key=Name,Value=lab-nat-a

NAT_B=$(aws ec2 create-nat-gateway \
  --subnet-id $PUB_SUB_B \
  --allocation-id $EIP_B \
  --query 'NatGateway.NatGatewayId' --output text)
aws ec2 create-tags --resources $NAT_B --tags Key=Name,Value=lab-nat-b

echo "Waiting for NAT Gateways to become available (~60s)..."
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_A $NAT_B
echo "NAT Gateways ready."
```

### Step 2.5 – Route Tables

```bash
# --- Public Route Table ---
PUB_RTB=$(aws ec2 create-route-table --vpc-id $VPC_ID \
  --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-tags --resources $PUB_RTB --tags Key=Name,Value=lab-public-rtb
# Route to internet
aws ec2 create-route --route-table-id $PUB_RTB \
  --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
# Associate both public subnets
aws ec2 associate-route-table --route-table-id $PUB_RTB --subnet-id $PUB_SUB_A
aws ec2 associate-route-table --route-table-id $PUB_RTB --subnet-id $PUB_SUB_B

# --- Private Route Table AZ-A ---
PRIV_RTB_A=$(aws ec2 create-route-table --vpc-id $VPC_ID \
  --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-tags --resources $PRIV_RTB_A --tags Key=Name,Value=lab-private-rtb-a
aws ec2 create-route --route-table-id $PRIV_RTB_A \
  --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_A
aws ec2 associate-route-table --route-table-id $PRIV_RTB_A --subnet-id $PRIV_SUB_A

# --- Private Route Table AZ-B ---
PRIV_RTB_B=$(aws ec2 create-route-table --vpc-id $VPC_ID \
  --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-tags --resources $PRIV_RTB_B --tags Key=Name,Value=lab-private-rtb-b
aws ec2 create-route --route-table-id $PRIV_RTB_B \
  --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_B
aws ec2 associate-route-table --route-table-id $PRIV_RTB_B --subnet-id $PRIV_SUB_B
```

### Step 2.6 – Security Groups

We create two security groups that enforce a strict traffic flow: Internet → ALB → EC2. Direct internet access to EC2 is blocked.

```bash
# --- ALB Security Group ---
ALB_SG=$(aws ec2 create-security-group \
  --group-name lab-alb-sg \
  --description "Allow HTTP from internet to ALB" \
  --vpc-id $VPC_ID \
  --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress \
  --group-id $ALB_SG \
  --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress \
  --group-id $ALB_SG \
  --protocol tcp --port 443 --cidr 0.0.0.0/0
aws ec2 create-tags --resources $ALB_SG --tags Key=Name,Value=lab-alb-sg

# --- EC2 Security Group ---
# Only allows traffic from the ALB security group — not from the internet directly
EC2_SG=$(aws ec2 create-security-group \
  --group-name lab-ec2-sg \
  --description "Allow HTTP only from ALB security group" \
  --vpc-id $VPC_ID \
  --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress \
  --group-id $EC2_SG \
  --protocol tcp --port 80 \
  --source-group $ALB_SG
aws ec2 create-tags --resources $EC2_SG --tags Key=Name,Value=lab-ec2-sg
```

> **🔒 Security Note:** EC2 instances have no public IP and no direct internet ingress. All web traffic flows exclusively through the ALB. SSH access is replaced by AWS Systems Manager Session Manager, eliminating the need to open port 22 entirely.

---

## 🪣 Module 3: S3 – Static Assets Bucket

**Goal:** Create a secure S3 bucket with a bucket policy, versioning, lifecycle rules, and encryption.

### Step 3.1 – Create the S3 Bucket

```bash
BUCKET_NAME="lab-assets-$(aws sts get-caller-identity --query Account --output text)"
echo "Bucket name: $BUCKET_NAME"

aws s3api create-bucket \
  --bucket $BUCKET_NAME \
  --region us-east-1
```

### Step 3.2 – Enable Versioning

Versioning protects against accidental deletion and enables rollback of assets.

```bash
aws s3api put-bucket-versioning \
  --bucket $BUCKET_NAME \
  --versioning-configuration Status=Enabled
```

### Step 3.3 – Block All Public Access

```bash
aws s3api put-public-access-block \
  --bucket $BUCKET_NAME \
  --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,\
    BlockPublicPolicy=true,RestrictPublicBuckets=true
```

### Step 3.4 – Apply a Bucket Policy

The bucket policy explicitly allows only the EC2 instance role to read objects. All other principals are denied. Create `bucket-policy.json`, replacing `ACCOUNT_ID` and `BUCKET_NAME` with your values:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2RoleAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:role/lab-ec2-instance-role"
      },
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME",
        "arn:aws:s3:::BUCKET_NAME/*"
      ]
    },
    {
      "Sid": "DenyAllOtherAccess",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::BUCKET_NAME",
        "arn:aws:s3:::BUCKET_NAME/*"
      ],
      "Condition": {
        "StringNotLike": {
          "aws:PrincipalArn": [
            "arn:aws:iam::ACCOUNT_ID:role/lab-ec2-instance-role",
            "arn:aws:iam::ACCOUNT_ID:root"
          ]
        }
      }
    }
  ]
}
```

```bash
# Apply the bucket policy (after editing the JSON with your Account ID and Bucket Name)
aws s3api put-bucket-policy \
  --bucket $BUCKET_NAME \
  --policy file://bucket-policy.json
```

> **💡 Key Concept:** Bucket policies are resource-based policies attached directly to the bucket. The `Deny` statement with a `StringNotLike` condition creates an explicit deny for all principals except the EC2 role and the root account — explicit denies always override allows in IAM evaluation logic.

### Step 3.5 – Configure Lifecycle Rules

Automatically transition old asset versions to cheaper storage and expire them after 90 days.

Create `lifecycle.json`:

```json
{
  "Rules": [
    {
      "ID": "transition-old-versions",
      "Status": "Enabled",
      "Filter": { "Prefix": "" },
      "NoncurrentVersionTransitions": [
        {
          "NoncurrentDays": 30,
          "StorageClass": "STANDARD_IA"
        }
      ],
      "NoncurrentVersionExpiration": {
        "NoncurrentDays": 90
      }
    },
    {
      "ID": "expire-incomplete-multipart",
      "Status": "Enabled",
      "Filter": { "Prefix": "" },
      "AbortIncompleteMultipartUpload": {
        "DaysAfterInitiation": 7
      }
    }
  ]
}
```

```bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket $BUCKET_NAME \
  --lifecycle-configuration file://lifecycle.json
```

### Step 3.6 – Enable Server-Side Encryption

```bash
aws s3api put-bucket-encryption \
  --bucket $BUCKET_NAME \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      },
      "BucketKeyEnabled": true
    }]
  }'
```

### Step 3.7 – Upload Sample Assets

```bash
# Create a sample CSS file
cat <<EOF > styles.css
body { font-family: Arial, sans-serif; background: #f4f4f4; text-align: center; padding: 40px; }
h1 { color: #FF9900; }
EOF

# Upload with correct content type and cache headers
aws s3 cp styles.css s3://$BUCKET_NAME/assets/styles.css \
  --content-type "text/css" \
  --cache-control "max-age=86400"

echo "Assets uploaded to s3://$BUCKET_NAME/assets/"
```

---

## ⚖️ Module 4: Application Load Balancer

**Goal:** Deploy an ALB in public subnets that distributes traffic to EC2 instances in private subnets.

### Step 4.1 – Create the ALB

```bash
ALB_ARN=$(aws elbv2 create-load-balancer \
  --name lab-alb \
  --subnets $PUB_SUB_A $PUB_SUB_B \
  --security-groups $ALB_SG \
  --scheme internet-facing \
  --type application \
  --ip-address-type ipv4 \
  --query 'LoadBalancers[0].LoadBalancerArn' \
  --output text)

echo "ALB ARN: $ALB_ARN"

# Get the ALB DNS name for testing later
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --load-balancer-arns $ALB_ARN \
  --query 'LoadBalancers[0].DNSName' \
  --output text)

echo "ALB DNS: $ALB_DNS"
```

### Step 4.2 – Create a Target Group

The Target Group defines how the ALB performs health checks and routes traffic to EC2 instances.

```bash
TG_ARN=$(aws elbv2 create-target-group \
  --name lab-target-group \
  --protocol HTTP \
  --port 80 \
  --vpc-id $VPC_ID \
  --target-type instance \
  --health-check-protocol HTTP \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3 \
  --matcher HttpCode=200 \
  --query 'TargetGroups[0].TargetGroupArn' \
  --output text)

echo "Target Group ARN: $TG_ARN"
```

### Step 4.3 – Create a Listener with Routing Rules

```bash
# Create HTTP listener on port 80
LISTENER_ARN=$(aws elbv2 create-listener \
  --load-balancer-arn $ALB_ARN \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=$TG_ARN \
  --query 'Listeners[0].ListenerArn' \
  --output text)

echo "Listener ARN: $LISTENER_ARN"
```

### Step 4.4 – Add a Custom Listener Rule

This rule returns a fixed `200 OK` for `/health` at the ALB level, independent of instance health.

```bash
aws elbv2 create-rule \
  --listener-arn $LISTENER_ARN \
  --priority 10 \
  --conditions '[{"Field":"path-pattern","Values":["/health"]}]' \
  --actions '[{"Type":"fixed-response","FixedResponseConfig":{"StatusCode":"200","ContentType":"text/plain","MessageBody":"OK"}}]'
```

> **💡 Production Note:** In a real environment, attach an ACM TLS certificate and create an HTTPS listener on port 443. Then add a redirect rule on port 80 that sends all HTTP traffic to HTTPS with a 301 status code.

---

## 📦 Module 5: EC2 Launch Template

**Goal:** Create a reusable, versioned Launch Template that the Auto Scaling Group will use to provision instances.

### Step 5.1 – Create a Key Pair (Optional — SSM is preferred)

```bash
aws ec2 create-key-pair \
  --key-name lab-key-pair \
  --query 'KeyMaterial' \
  --output text > lab-key-pair.pem

chmod 400 lab-key-pair.pem
```

### Step 5.2 – Fetch the Latest Amazon Linux 2023 AMI

```bash
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters 'Name=name,Values=al2023-ami-*-x86_64' \
            'Name=state,Values=available' \
  --query 'sort_by(Images, &CreationDate)[-1].ImageId' \
  --output text)

echo "Latest AMI: $AMI_ID"
```

### Step 5.3 – Write the User Data Script

Save this as `userdata.sh`:

```bash
#!/bin/bash
set -e
yum update -y
yum install -y httpd aws-cli

# Start Apache
systemctl start httpd
systemctl enable httpd

# Health check endpoint
echo "OK" > /var/www/html/health

# Pull static assets from S3
ACCOUNT_ID=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document \
  | python3 -c 'import sys,json; print(json.load(sys.stdin)["accountId"])')
BUCKET_NAME="lab-assets-${ACCOUNT_ID}"

mkdir -p /var/www/html/assets
aws s3 cp s3://${BUCKET_NAME}/assets/styles.css /var/www/html/assets/styles.css || true

# Retrieve instance metadata
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)

# Deploy the app page
cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
  <title>AWS Lab App</title>
  <link rel="stylesheet" href="/assets/styles.css">
</head>
<body>
  <h1>🚀 Production-Grade AWS Application</h1>
  <p><strong>Instance ID:</strong> ${INSTANCE_ID}</p>
  <p><strong>Availability Zone:</strong> ${AZ}</p>
  <p>Traffic routed via ALB &rarr; Auto Scaling Group &rarr; EC2 (private subnet)</p>
</body>
</html>
EOF
```

### Step 5.4 – Create the Launch Template

```bash
USER_DATA=$(base64 -w 0 userdata.sh)
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

LT_ID=$(aws ec2 create-launch-template \
  --launch-template-name lab-launch-template \
  --version-description "v1-initial" \
  --launch-template-data "{
    \"ImageId\": \"$AMI_ID\",
    \"InstanceType\": \"t3.micro\",
    \"IamInstanceProfile\": {
      \"Name\": \"lab-ec2-profile\"
    },
    \"SecurityGroupIds\": [\"$EC2_SG\"],
    \"UserData\": \"$USER_DATA\",
    \"TagSpecifications\": [{
      \"ResourceType\": \"instance\",
      \"Tags\": [{\"Key\":\"Name\",\"Value\":\"lab-web-server\"}]
    }],
    \"MetadataOptions\": {
      \"HttpTokens\": \"required\",
      \"HttpPutResponseHopLimit\": 1
    },
    \"Monitoring\": {\"Enabled\": true}
  }" \
  --query 'LaunchTemplate.LaunchTemplateId' \
  --output text)

echo "Launch Template ID: $LT_ID"
```

> **🔒 Security Note:** `HttpTokens: required` enforces IMDSv2, which prevents SSRF attacks from abusing the instance metadata service. Always set this in production.

---

## 📈 Module 6: Auto Scaling Group

**Goal:** Deploy an ASG that maintains availability across AZs and scales automatically based on CPU and request load.

### Step 6.1 – Create the Auto Scaling Group

```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name lab-asg \
  --launch-template "LaunchTemplateId=$LT_ID,Version=\$Latest" \
  --min-size 2 \
  --max-size 6 \
  --desired-capacity 2 \
  --vpc-zone-identifier "$PRIV_SUB_A,$PRIV_SUB_B" \
  --target-group-arns $TG_ARN \
  --health-check-type ELB \
  --health-check-grace-period 120 \
  --tags "Key=Name,Value=lab-asg-instance,PropagateAtLaunch=true" \
         "Key=Environment,Value=lab,PropagateAtLaunch=true"
```

### Step 6.2 – Configure Target Tracking Scaling Policy (CPU)

Automatically scale out when average CPU exceeds 60%, and scale in when it drops.

```bash
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name lab-asg \
  --policy-name lab-cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 60.0,
    "ScaleInCooldown": 300,
    "ScaleOutCooldown": 60
  }'
```

### Step 6.3 – Configure Request Count Scaling Policy (ALB)

Scale based on the number of requests per target — useful for I/O-bound or latency-sensitive applications.

```bash
# Get the ALB suffix for the resource label
ALB_SUFFIX=$(echo $ALB_ARN | cut -d'/' -f2-4)
TG_SUFFIX=$(echo $TG_ARN | cut -d'/' -f2-4)

aws autoscaling put-scaling-policy \
  --auto-scaling-group-name lab-asg \
  --policy-name lab-request-count-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration "{
    \"PredefinedMetricSpecification\": {
      \"PredefinedMetricType\": \"ALBRequestCountPerTarget\",
      \"ResourceLabel\": \"${ALB_SUFFIX}/targetgroup/${TG_SUFFIX}\"
    },
    \"TargetValue\": 1000.0,
    \"ScaleInCooldown\": 300,
    \"ScaleOutCooldown\": 60
  }"
```

### Step 6.4 – Configure Scheduled Scaling Actions

Pre-warm your fleet before a known traffic spike (e.g., business hours).

```bash
# Scale up at 8am UTC on weekdays
aws autoscaling put-scheduled-action \
  --auto-scaling-group-name lab-asg \
  --scheduled-action-name scale-up-business-hours \
  --recurrence "0 8 * * MON-FRI" \
  --min-size 4 \
  --max-size 6 \
  --desired-capacity 4

# Scale back down at 8pm UTC on weekdays
aws autoscaling put-scheduled-action \
  --auto-scaling-group-name lab-asg \
  --scheduled-action-name scale-down-off-hours \
  --recurrence "0 20 * * MON-FRI" \
  --min-size 2 \
  --max-size 6 \
  --desired-capacity 2
```

### Step 6.5 – Enable Instance Refresh

Instance Refresh allows you to roll out a new Launch Template version (e.g., new AMI or updated user data) across the ASG with zero downtime using a rolling replacement strategy.

```bash
# First create a new Launch Template version (e.g., with an updated AMI or config change)
aws ec2 create-launch-template-version \
  --launch-template-id $LT_ID \
  --version-description "v2-updated" \
  --source-version 1 \
  --launch-template-data '{"InstanceType":"t3.small"}'

# Trigger a rolling instance refresh targeting the new version
aws autoscaling start-instance-refresh \
  --auto-scaling-group-name lab-asg \
  --preferences '{
    "MinHealthyPercentage": 80,
    "InstanceWarmup": 120,
    "CheckpointPercentages": [33, 66, 100],
    "CheckpointDelay": 120
  }'

# Monitor the refresh status
aws autoscaling describe-instance-refreshes \
  --auto-scaling-group-name lab-asg \
  --query 'InstanceRefreshes[0].{Status:Status,PercentComplete:PercentageComplete}'
```

---

## ✅ Module 7: Validation and Testing

### Step 7.1 – Verify the ALB and Instances

```bash
# Check instance registration in the target group
aws elbv2 describe-target-health \
  --target-group-arn $TG_ARN \
  --query 'TargetHealthDescriptions[*].{Id:Target.Id,State:TargetHealth.State}'

# Wait for all instances to pass health checks
echo "Waiting for targets to become healthy..."
while true; do
  HEALTHY=$(aws elbv2 describe-target-health \
    --target-group-arn $TG_ARN \
    --query 'length(TargetHealthDescriptions[?TargetHealth.State==`healthy`])' \
    --output text)
  echo "Healthy targets: $HEALTHY / 2"
  [ "$HEALTHY" -ge 2 ] && break
  sleep 15
done
echo "All targets healthy!"
```

### Step 7.2 – Test the Application

```bash
# Test the load balancer
curl http://$ALB_DNS/
curl http://$ALB_DNS/health

# Confirm ALB is distributing requests across AZs (notice different Instance IDs)
echo "Checking load distribution across AZs:"
for i in {1..10}; do
  curl -s http://$ALB_DNS/ | grep "Instance ID\|Availability Zone"
  echo "---"
done
```

### Step 7.3 – Test S3 Asset Delivery via EC2

```bash
curl http://$ALB_DNS/assets/styles.css
```

### Step 7.4 – Connect to an Instance via SSM (No SSH Required)

```bash
# Get an instance ID from the ASG
INSTANCE_ID=$(aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names lab-asg \
  --query 'AutoScalingGroups[0].Instances[0].InstanceId' \
  --output text)

echo "Connecting to: $INSTANCE_ID"

# Open a secure shell session via SSM — no key pair or open port 22 needed
aws ssm start-session --target $INSTANCE_ID
```

### Step 7.5 – Stress Test and Watch Auto Scaling

Inside the SSM session, run a CPU stress test:

```bash
# Install and run stress tool (inside the SSM session)
sudo yum install -y stress
stress --cpu 4 --timeout 300 &
exit
```

Back in your local terminal, watch the ASG respond:

```bash
# Monitor scaling activity in real time
watch -n 10 "aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name lab-asg \
  --query 'Activities[0:5].{Status:StatusCode,Desc:Description}' \
  --output table"

# Watch instance count change
watch -n 10 "aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names lab-asg \
  --query 'AutoScalingGroups[0].{Min:MinSize,Max:MaxSize,Desired:DesiredCapacity,Running:length(Instances)}'"
```

---

## 🧪 Lab Validation Checklist

**Module 1 – IAM**
- [ ] EC2 instance policy created with S3, CloudWatch Logs, and SSM permissions
- [ ] Permission boundary created and attached to the EC2 role
- [ ] EC2 trust policy scoped to `ec2.amazonaws.com`
- [ ] Instance profile `lab-ec2-profile` created and role attached
- [ ] Lab deploy user created and added to `lab-deploy-group`

**Module 2 – VPC**
- [ ] VPC `lab-vpc` created with CIDR `10.0.0.0/16`, DNS hostnames enabled
- [ ] 2 public and 2 private subnets created across AZ-A and AZ-B
- [ ] Internet Gateway attached; public route table routes `0.0.0.0/0` → IGW
- [ ] NAT Gateways deployed in both public subnets (one per AZ)
- [ ] Private route tables route `0.0.0.0/0` → respective NAT Gateway
- [ ] ALB Security Group allows ports 80/443 from `0.0.0.0/0`
- [ ] EC2 Security Group allows port 80 **only from the ALB SG** (not from internet)

**Module 3 – S3**
- [ ] Bucket created with all public access blocked
- [ ] Versioning enabled on the bucket
- [ ] Bucket policy allows EC2 role only; denies all other principals
- [ ] Server-side encryption enabled (AES-256 with Bucket Key)
- [ ] Lifecycle rule transitions old versions to S3-IA at 30 days, expires at 90
- [ ] Sample CSS asset uploaded successfully

**Module 4 – ALB**
- [ ] ALB deployed in both public subnets (internet-facing)
- [ ] Target group created with `/health` health check path (HTTP 200 required)
- [ ] HTTP listener on port 80 forwards traffic to the target group
- [ ] Custom listener rule returns fixed 200 for `/health` at ALB level

**Module 5 – Launch Template**
- [ ] Launch Template uses latest Amazon Linux 2023 AMI
- [ ] IAM instance profile `lab-ec2-profile` attached
- [ ] IMDSv2 enforced (`HttpTokens: required`)
- [ ] User Data installs Apache, creates `/health`, and pulls assets from S3
- [ ] Detailed monitoring enabled

**Module 6 – Auto Scaling**
- [ ] ASG spans both private subnets across 2 AZs
- [ ] Min=2, Max=6, Desired=2 configured
- [ ] ELB health checks with 120-second grace period
- [ ] CPU target tracking policy at 60% threshold
- [ ] ALB request count tracking policy at 1000 req/target
- [ ] Scheduled scale-up (8am) and scale-down (8pm) weekday actions created
- [ ] Instance Refresh successfully rolled out updated Launch Template version

**Module 7 – Validation**
- [ ] All 2 targets healthy in the target group
- [ ] `curl http://<ALB-DNS>/` returns the HTML app page
- [ ] `curl http://<ALB-DNS>/health` returns `OK`
- [ ] Multiple requests show different Instance IDs/AZs (load balancing confirmed)
- [ ] S3 CSS asset accessible via `http://<ALB-DNS>/assets/styles.css`
- [ ] SSM Session Manager connection to a private EC2 instance successful (no SSH)
- [ ] ASG scaled out during CPU stress test; scaled back in after cooldown

---

## 🧹 Cleanup – Avoid Unexpected Charges

> ⚠️ **Follow this order precisely.** Deleting resources out of order causes dependency errors.

```bash
# 1. Delete Scheduled Scaling Actions
aws autoscaling delete-scheduled-action --auto-scaling-group-name lab-asg --scheduled-action-name scale-up-business-hours
aws autoscaling delete-scheduled-action --auto-scaling-group-name lab-asg --scheduled-action-name scale-down-off-hours

# 2. Delete Scaling Policies
aws autoscaling delete-policy --auto-scaling-group-name lab-asg --policy-name lab-cpu-target-tracking
aws autoscaling delete-policy --auto-scaling-group-name lab-asg --policy-name lab-request-count-tracking

# 3. Delete Auto Scaling Group (terminates all EC2 instances)
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name lab-asg --force-delete
echo "Waiting 60s for instances to terminate..."
sleep 60

# 4. Delete Launch Template (all versions)
aws ec2 delete-launch-template --launch-template-id $LT_ID

# 5. Delete ALB Listener, then Load Balancer
aws elbv2 delete-listener --listener-arn $LISTENER_ARN
aws elbv2 delete-load-balancer --load-balancer-arn $ALB_ARN

# 6. Delete Target Group
aws elbv2 delete-target-group --target-group-arn $TG_ARN

# 7. Empty and delete S3 bucket (must empty versioned bucket first)
aws s3api list-object-versions --bucket $BUCKET_NAME \
  --query '{Objects: Versions[].{Key:Key,VersionId:VersionId}}' \
  --output json | python3 -c "
import json, sys, subprocess
data = json.load(sys.stdin)
for obj in data.get('Objects', []):
    subprocess.run(['aws', 's3api', 'delete-object', '--bucket', '$BUCKET_NAME',
                    '--key', obj['Key'], '--version-id', obj['VersionId']])
"
aws s3api delete-bucket --bucket $BUCKET_NAME

# 8. Delete NAT Gateways
aws ec2 delete-nat-gateway --nat-gateway-id $NAT_A
aws ec2 delete-nat-gateway --nat-gateway-id $NAT_B
echo "Waiting for NAT Gateways to delete (~60s)..."
aws ec2 wait nat-gateway-deleted --nat-gateway-ids $NAT_A $NAT_B

# 9. Release Elastic IPs
aws ec2 release-address --allocation-id $EIP_A
aws ec2 release-address --allocation-id $EIP_B

# 10. Delete Security Groups
aws ec2 delete-security-group --group-id $EC2_SG
aws ec2 delete-security-group --group-id $ALB_SG

# 11. Delete Subnets
for SUBNET in $PUB_SUB_A $PUB_SUB_B $PRIV_SUB_A $PRIV_SUB_B; do
  aws ec2 delete-subnet --subnet-id $SUBNET
done

# 12. Delete Route Tables
for RTB in $PUB_RTB $PRIV_RTB_A $PRIV_RTB_B; do
  aws ec2 delete-route-table --route-table-id $RTB
done

# 13. Detach and Delete Internet Gateway
aws ec2 detach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID

# 14. Delete VPC
aws ec2 delete-vpc --vpc-id $VPC_ID

# 15. Delete IAM Resources
aws iam remove-role-from-instance-profile --instance-profile-name lab-ec2-profile --role-name lab-ec2-instance-role
aws iam delete-instance-profile --instance-profile-name lab-ec2-profile
aws iam detach-role-policy --role-name lab-ec2-instance-role --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/lab-ec2-instance-policy
aws iam delete-role --role-name lab-ec2-instance-role
aws iam delete-policy --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/lab-ec2-instance-policy
aws iam delete-policy --policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/lab-ec2-permission-boundary
aws iam remove-user-from-group --group-name lab-deploy-group --user-name lab-deploy-user
for POLICY in AmazonEC2FullAccess AmazonS3FullAccess ElasticLoadBalancingFullAccess AutoScalingFullAccess; do
  aws iam detach-group-policy --group-name lab-deploy-group --policy-arn arn:aws:iam::aws:policy/$POLICY
done
aws iam delete-group --group-name lab-deploy-group
aws iam delete-user --user-name lab-deploy-user

# 16. Delete Key Pair
aws ec2 delete-key-pair --key-name lab-key-pair
rm -f lab-key-pair.pem userdata.sh styles.css ec2-policy.json permission-boundary.json \
      ec2-trust-policy.json bucket-policy.json lifecycle.json

echo "✅ Cleanup complete. All lab resources have been deleted."
```

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
| IMDSv2 (Metadata Service) | https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html |
| SSM Session Manager | https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html |

---

## 📝 Lab Summary

| Module | Service | Key Concepts Covered |
|---|---|---|
| 1 | IAM | Roles, inline vs managed policies, permission boundaries, instance profiles, least privilege |
| 2 | VPC | Multi-AZ design, public/private subnets, IGW, NAT Gateways, route tables, security group chaining |
| 3 | S3 | Bucket policies, resource-based vs identity-based policies, versioning, lifecycle rules, SSE-S3 encryption |
| 4 | ALB | Target groups, health checks, listener rules, fixed-response actions, multi-AZ distribution |
| 5 | EC2 | Launch Templates, IMDSv2, user data bootstrapping, SSM access (no SSH) |
| 6 | Auto Scaling | Target tracking (CPU + ALB), scheduled scaling, instance refresh with checkpoints |
| 7 | Validation | SSM Session Manager, CPU stress testing, real-time ASG scaling observation |

---

*Built with AWS best practices in mind. Happy building! ☁️*
