

## 🧪 AWS Hands-On Lab: Host a Website on EC2 with ASG, SNS & S3
<img width="1440" height="890" alt="image" src="https://github.com/user-attachments/assets/2a0244d3-14f8-4a89-a530-cd461f97f95c" />


---

## Phase 1 — S3 Bucket (Static Assets)

**Step 1.** Go to S3 → **Create bucket**
- Name: `my-website-assets-<yourname>`
- Region: `us-east-1`
- Uncheck "Block all public access" → acknowledge
- Leave versioning off → **Create**

**Step 2.** Upload your site files (create a simple `index.html` locally):
```html
<!DOCTYPE html>
<html>
<head><title>My EC2 Website</title></head>
<body>
  <h1>Hello from EC2!</h1>
  <p>Hosted behind an Auto Scaling Group.</p>
</body>
</html>
```

Upload `index.html` to the bucket. Note the bucket name — EC2 instances will pull from it via IAM role.

---

## Phase 2 — IAM Role for EC2

**Step 3.** Go to IAM → **Roles** → **Create role**
- Trusted entity: `EC2`
- Attach policy: `AmazonS3ReadOnlyAccess`
- Name: `EC2-S3-ReadOnly-Role` → **Create**

---

## Phase 3 — SNS Topic for Notifications

**Step 4.** Go to SNS → **Topics** → **Create topic**
- Type: `Standard`
- Name: `ASG-Scale-Alerts`
- **Create topic**

**Step 5.** Create a subscription:
- Protocol: `Email`
- Endpoint: your email address → **Create subscription**
- Check your email and **confirm the subscription**

---

## Phase 4 — Launch Template

**Step 6.** Go to EC2 → **Launch Templates** → **Create launch template**
- Name: `web-server-template`
- AMI: `Amazon Linux 2023` (latest)
- Instance type: `t2.micro`
- Key pair: either go with `proceed without keypair` or create or select already existing one
- Security Group: create new with these inbound rules:
  - SSH (22) — your IP
  - HTTP (80) — Anywhere (0.0.0.0/0)
- IAM instance profile: `EC2-S3-ReadOnly-Role`

**Step 7.** Under **Advanced details → User data**, paste:
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd

# Pull index.html from your S3 bucket
aws s3 cp s3://my-website-assets-<yourname>/index.html /var/www/html/index.html

echo "Instance ID: $(curl -s http://169.254.169.254/latest/meta-data/instance-id)" >> /var/www/html/index.html
```

Click **Create launch template**.

---

## Phase 5 — Application Load Balancer

**Step 8.** Go to EC2 → **Load Balancers** → **Create Load Balancer** → choose **Application Load Balancer**
- Name: `web-alb`
- Scheme: Internet-facing
- Subnets: select at least **2 Availability Zones**

**Step 9.** Create a **Target Group**:
- Target type: `Instances`
- Name: `web-tg`
- Protocol: HTTP, Port: 80
- Health check path: `/`
- **Create target group** (leave empty — ASG will register targets)

**Step 10.** Back in ALB creation, attach the target group → **Create load balancer**

---

## Phase 6 — Auto Scaling Group

**Step 11.** Go to EC2 → **Auto Scaling Groups** → **Create Auto Scaling group**
- Name: `web-asg`
- Launch template: `web-server-template` (latest version)

**Step 12.** Configure networking:
- VPC: default
- Subnets: select 2+ subnets across AZs

**Step 13.** Attach to load balancer:
- Select **Attach to an existing load balancer**
- Choose your target group `web-tg`
- Enable **ELB health checks**

**Step 14.** Set capacity:
| Setting | Value |
|---|---|
| Desired | 2 |
| Minimum | 1 |
| Maximum | 4 |

**Step 15.** Configure scaling policies:
- Add policy: **Target tracking scaling**
- Metric: `Average CPU Utilization`
- Target value: `50`

**Step 16.** Add notifications (SNS):
- Click **Add notification**
- SNS topic: `ASG-Scale-Alerts`
- Events to notify: ✅ Launch ✅ Terminate ✅ Launch error ✅ Terminate error
- **Create Auto Scaling group**

---

## Phase 7 — Test Everything

**Step 17.** Wait 3–5 minutes for instances to launch and pass health checks.

**Step 18.** Go to **Load Balancers** → copy the **DNS name** → paste in browser. You should see your webpage with the instance ID at the bottom.

**Step 19.** Simulate a scale-out event to test SNS:
```bash
# Either Directly connect or SSH into one of the EC2 instances, then:

sudo yum install -y stress
stress --cpu 4 --timeout 300
```
After ~5 minutes, watch the ASG spin up a new instance. You'll receive an email from SNS notifying you of the launch event.

**Step 20.** Verify scale-in: once the stress test ends, CPU drops, and after the cooldown period (default 300s), the ASG terminates the extra instance — another SNS email arrives.

---

## Phase 8 — Cleanup (avoid charges)

Delete resources in this order to avoid dependency errors:

1. Auto Scaling Group → set desired/min/max to 0, then delete
2. Load Balancer → delete
3. Target Group → delete
4. Launch Template → delete
5. SNS Subscription & Topic → delete
6. S3 Bucket → empty first, then delete
7. IAM Role → delete

---
