# AWS Elastic Beanstalk

## 📘 Introduction

AWS Elastic Beanstalk is a **Platform as a Service (PaaS)** provided by AWS that helps developers deploy, manage, and scale applications without manually managing infrastructure.

You simply upload your application code, and Elastic Beanstalk automatically handles:

- EC2 instance provisioning
- Load balancing
- Auto Scaling
- Monitoring
- Health checks
- Application deployment

---

# 📋 Table of Contents

- [What is Elastic Beanstalk?](#-what-is-elastic-beanstalk)
- [Key Features](#-key-features)
- [Core Components](#-core-components)
- [Architecture](#-architecture)
- [Deployment Workflow](#-deployment-workflow)
- [Environment Types](#-environment-types)
- [Deployment Policies](#-deployment-policies)
- [Docker Support](#-docker-support)
- [AWS Services Used](#-aws-services-used)
- [Configuration Files](#-configuration-files)
- [EB CLI Commands](#-eb-cli-commands)
- [Scaling](#-scaling)
- [Security](#-security)
- [Database Options](#-database-options)
- [Advantages](#-advantages)
- [Disadvantages](#-disadvantages)
- [Elastic Beanstalk vs Other Services](#-elastic-beanstalk-vs-other-services)
- [Interview Questions](#-interview-questions)
- [Best Practices](#-best-practices)
- [Real-World Use Cases](#-real-world-use-cases)
- [Conclusion](#-conclusion)

---

# 🚀 What is Elastic Beanstalk?

Elastic Beanstalk is a service that allows developers to deploy applications quickly without worrying about the underlying infrastructure.

Supported platforms include:

- Python
- Node.js
- Java
- PHP
- Ruby
- Go
- .NET
- Docker

---

# ✨ Key Features

## 1. Easy Deployment

Deploy applications using:

- ZIP file upload
- Git
- CI/CD pipelines
- Docker containers

---

## 2. Auto Scaling

Automatically increases or decreases EC2 instances based on traffic.

---

## 3. Load Balancing

Distributes incoming traffic across multiple EC2 instances.

---

## 4. Health Monitoring

Provides application and instance health monitoring.

Health statuses:

- Green → Healthy
- Yellow → Warning
- Red → Critical

---

## 5. CloudWatch Integration

Monitors:

- CPU utilization
- Memory usage
- Request count
- Network traffic

---

# 🧩 Core Components

## 1. Application

Top-level container for Elastic Beanstalk resources.

Example:

```text
ecommerce-app
```

---

## 2. Environment

Actual running infrastructure.

Examples:

```text
Dev
Test
Production
```

Contains:

- EC2 instances
- Load balancer
- Auto Scaling group

---

## 3. Environment Tier

### Web Server Environment

Used for:

- Websites
- APIs
- Web applications

---

### Worker Environment

Used for:

- Background jobs
- Queue processing

Works with SQS.

---

## 4. Application Version

Each uploaded code package becomes a version.

Example:

```text
v1
v2
v3
```

---

## 5. Platform

Defines runtime environment.

Examples:

- Python 3.12
- Node.js
- Docker
- Tomcat

---

# 🏗️ Architecture

```text
Users
   ↓
Load Balancer
   ↓
EC2 Instances
   ↓
Application
```

Behind the scenes Elastic Beanstalk creates:

- EC2
- Auto Scaling Group
- Application Load Balancer
- Security Groups
- CloudWatch Alarms

---

# 🔄 Deployment Workflow

## Step 1 — Create Application

```text
Application Name: ecommerce-app
```

---

## Step 2 — Create Environment

Choose:

- Web Server
- Worker Environment

---

## Step 3 — Select Platform

Example:

```text
Python / Node.js / Docker
```

---

## Step 4 — Upload Code

Upload:

```text
ZIP File
```

Or connect:

- GitHub
- CodePipeline

---

## Step 5 — AWS Creates Resources

Elastic Beanstalk automatically provisions:

- EC2
- ALB
- Auto Scaling
- Security Groups

---

## Step 6 — Access Application

AWS provides URL:

```text
http://application-name.region.elasticbeanstalk.com
```

---

# 🌍 Environment Types

## 1. Single Instance Environment

Architecture:

```text
User → EC2
```

Used for:

- Development
- Testing

No load balancer.

---

## 2. Load Balanced Environment

Architecture:

```text
User → ALB → Multiple EC2 Instances
```

Used for:

- Production

Supports:

- Auto Scaling
- High Availability

---

# 🚢 Deployment Policies

## 1. All at Once

Updates all instances simultaneously.

### Pros

- Fast deployment

### Cons

- Possible downtime

---

## 2. Rolling

Updates instances in batches.

### Pros

- Reduced downtime

### Cons

- Slower deployment

---

## 3. Rolling with Additional Batch

Creates temporary instances during deployment.

### Pros

- Better availability

### Cons

- Higher cost

---

## 4. Immutable Deployment

Creates entirely new instances.

### Pros

- Safest deployment

### Cons

- Expensive

---

## 5. Blue/Green Deployment

Two separate environments:

- Blue → Current version
- Green → New version

Traffic switches after testing.

### Pros

- Near-zero downtime
- Easy rollback

### Cons

- Temporary double cost

---

# 🐳 Docker Support

Elastic Beanstalk supports:

- Single-container Docker
- Multi-container Docker

Deploy using:

```text
Dockerfile
docker-compose.yml
```

---

# ☁️ AWS Services Used

Elastic Beanstalk internally uses:

| Service | Purpose |
|---|---|
| EC2 | Runs application |
| ALB | Load balancing |
| Auto Scaling | Scaling instances |
| CloudWatch | Monitoring |
| IAM | Permissions |
| S3 | Stores application versions |
| RDS | Database integration |

---

# ⚙️ Configuration Files

## `.ebextensions`

Used for:

- Custom configurations
- Environment variables
- Package installations

Example:

```yaml
option_settings:
  aws:autoscaling:launchconfiguration:
    InstanceType: t3.micro
```

---

# 🖥️ EB CLI Commands

## Install EB CLI

```bash
pip install awsebcli
```

---

## Initialize Project

```bash
eb init
```

---

## Create Environment

```bash
eb create
```

---

## Deploy Application

```bash
eb deploy
```

---

## Open Application

```bash
eb open
```

---

## View Logs

```bash
eb logs
```

---

## SSH into Instance

```bash
eb ssh
```

---

## Terminate Environment

```bash
eb terminate
```

---

# 📈 Scaling

Elastic Beanstalk supports Auto Scaling.

Example configuration:

```text
Minimum Instances: 2
Maximum Instances: 6
Scale Out at 70% CPU
```

---

# 🔐 Security

## IAM Roles

Used for:

- EC2 permissions
- Elastic Beanstalk service permissions

---

## Security Groups

Controls:

- Inbound traffic
- Outbound traffic

Example:

```text
80 → HTTP
443 → HTTPS
22 → SSH
```

---

# 🗄️ Database Options

## Option 1 — RDS Inside Elastic Beanstalk

### Pros

- Easy setup

### Cons

- Database may be deleted with environment

---

## Option 2 — Separate RDS (Recommended)

### Pros

- Persistent database
- Safer

### Cons

- Slightly more setup

---

# ✅ Advantages

- Easy deployment
- Managed infrastructure
- Built-in monitoring
- Auto Scaling support
- Supports CI/CD
- Beginner-friendly

---

# ❌ Disadvantages

- Less infrastructure control
- Not ideal for advanced microservices
- Hidden AWS resource creation
- Limited orchestration compared to Kubernetes

---

# ⚔️ Elastic Beanstalk vs Other Services

## Elastic Beanstalk vs EC2

| Feature | EC2 | Elastic Beanstalk |
|---|---|---|
| Infrastructure | Manual | Automatic |
| Scaling | Manual | Automatic |
| Load Balancer | Manual | Automatic |
| Monitoring | Manual | Built-in |

---

## Elastic Beanstalk vs ECS

| Feature | Elastic Beanstalk | ECS |
|---|---|---|
| Complexity | Easy | Medium |
| Container Orchestration | Limited | Advanced |
| Best For | Simple Apps | Microservices |

---

## Elastic Beanstalk vs Lambda

| Feature | Elastic Beanstalk | Lambda |
|---|---|---|
| Server Management | Managed | Serverless |
| Runtime | Long-running | Short-lived |
| Best For | Web Apps | Event-driven tasks |

---

# 🎯 Interview Questions

## What is Elastic Beanstalk?

A PaaS service that automates application deployment and infrastructure management.

---

## Does Elastic Beanstalk use EC2 internally?

Yes.

---

## Can we SSH into Elastic Beanstalk instances?

Yes.

```bash
eb ssh
```

---

## Does Elastic Beanstalk support Docker?

Yes.

Supports:

- Single-container Docker
- Multi-container Docker

---

## What happens during high traffic?

Elastic Beanstalk uses Auto Scaling to launch additional EC2 instances.

---

# 🏢 Real-World Use Cases

## E-Commerce Applications

Auto scales during sales traffic.

---

## REST APIs

Deploy:

- Flask APIs
- Node.js APIs
- Spring Boot applications

---

## Startup MVPs

Quick deployment with minimal infrastructure management.

---

## Internal Company Portals

Simple hosting and monitoring.

---

# 💡 Best Practices

- Use separate RDS database
- Enable Auto Scaling
- Use Blue/Green deployment
- Store secrets in Secrets Manager
- Enable HTTPS using ACM
- Monitor with CloudWatch

---

# 🧠 Important Tips

Remember:

- Elastic Beanstalk = PaaS
- Uses EC2 internally
- Supports Auto Scaling
- Supports Load Balancers
- Supports Docker
- Supports Blue/Green deployments

---

