
# What is AWS?

Amazon Web Services (AWS) is a comprehensive cloud computing platform provided by Amazon. It offers over 200 fully featured services from data centers globally, including computing power, storage, databases, networking, analytics, machine learning, and more.

**Key Benefits:**
- Pay-as-you-go pricing model
- Scalability and flexibility
- Global infrastructure
- High reliability and security
- No upfront infrastructure costs

## Creating a Free Tier Account

The AWS Free Tier provides limited access to AWS services for 12 months at no cost, perfect for learning and experimentation.

**Steps to Create an Account:**

1. **Visit the AWS Website:** Go to aws.amazon.com and click "Create an AWS Account"

2. **Provide Account Information:**
   - Enter your email address
   - Create a password
   - Choose an AWS account name

3. **Contact Information:**
   - Select account type (Personal or Professional)
   - Enter your full name, phone number, and address

4. **Payment Information:**
   - Add a valid credit/debit card (required for identity verification)
   - You won't be charged unless you exceed Free Tier limits

5. **Identity Verification:**
   - Verify your identity via phone call or SMS
   - Enter the verification code received

6. **Select Support Plan:**
   - Choose the Basic Support Plan (free)
   - Other plans available: Developer, Business, Enterprise

7. **Complete Sign-up:**
   - Review and confirm your information
   - Access the AWS Management Console

**Free Tier Includes:**
- 750 hours per month of EC2 t2.micro or t3.micro instances
- 5 GB of S3 storage
- 750 hours of RDS database usage
- 25 GB of DynamoDB storage
- Many other services with usage limits

## Core AWS Services

AWS offers services across multiple categories. Here are the essential ones:

**Compute Services:**
- **EC2 (Elastic Compute Cloud):** Virtual servers in the cloud that you can configure and scale
- **Lambda:** Serverless computing that runs code in response to events without managing servers
- **Elastic Beanstalk:** Platform for deploying and scaling web applications automatically

**Storage Services:**
- **S3 (Simple Storage Service):** Object storage for any type of data with high durability
- **EBS (Elastic Block Store):** Persistent block storage for EC2 instances
- **Glacier:** Low-cost archival storage for infrequently accessed data
- **EFS (Elastic File System):** Scalable file storage for EC2 instances

**Database Services:**
- **RDS (Relational Database Service):** Managed relational databases including MySQL, PostgreSQL, Oracle, SQL Server
- **DynamoDB:** Fully managed NoSQL database with single-digit millisecond latency
- **ElastiCache:** In-memory caching service using Redis or Memcached
- **Redshift:** Data warehouse for analytics and big data

**Networking Services:**
- **VPC (Virtual Private Cloud):** Isolated network environment within AWS
- **Route 53:** Scalable DNS and domain registration service
- **CloudFront:** Content Delivery Network for fast content distribution
- **Direct Connect:** Dedicated network connection from your premises to AWS

**Security and Identity:**
- **IAM (Identity and Access Management):** Control access to AWS resources
- **KMS (Key Management Service):** Create and manage encryption keys
- **AWS Shield:** DDoS protection
- **WAF (Web Application Firewall):** Protect web applications from common exploits

**Management and Monitoring:**
- **CloudWatch:** Monitoring and observability service
- **CloudFormation:** Infrastructure as code for provisioning resources
- **CloudTrail:** Log and monitor account activity
- **Systems Manager:** Operational insights and automated actions

## AWS Global Partitions

AWS infrastructure is divided into separate partitions, each representing an independent deployment of AWS services with its own endpoints and resources.

### 1. **AWS Standard (Global) Partition**

The primary AWS partition serving most of the world.

- **Partition Name:** `aws`
- **Regions:** Includes all standard commercial regions worldwide
- **Examples:** us-east-1, eu-west-1, ap-southeast-1
- **Endpoint Format:** service.region.amazonaws.com
- **Use Case:** General commercial use globally

### 2. **AWS China Partition**

A separate partition operated by local Chinese companies due to regulatory requirements.

- **Partition Name:** `aws-cn`
- **Operator:** Operated by Sinnet (Beijing) and NWCD (Ningxia)
- **Regions:** cn-north-1 (Beijing), cn-northwest-1 (Ningxia)
- **Endpoint Format:** service.region.amazonaws.com.cn
- **Key Differences:**
  - Requires separate account from standard AWS
  - Different billing and support structure
  - Compliance with Chinese data sovereignty laws
  - Limited service availability compared to global partition
  - ICP license required for hosting websites

### 3. **AWS GovCloud (US) Partition**

Isolated regions designed for US government agencies and contractors.

- **Partition Name:** `aws-us-gov`
- **Regions:** us-gov-west-1, us-gov-east-1
- **Endpoint Format:** service.region.amazonaws-us-gov.com
- **Key Features:**
  - FedRAMP High compliance
  - ITAR (International Traffic in Arms Regulations) compliance
  - Operated by US citizens on US soil
  - Physically isolated from standard AWS regions
  - Requires vetting process for access
  - Supports classified workloads up to Secret level

**Important Notes About Partitions:**
- Resources in one partition cannot directly interact with resources in another partition
- IAM credentials are partition-specific
- Services and features may vary between partitions
- Separate billing and account management for each partition

## Core AWS Concepts (In-Depth)

### Regions

A **Region** is a physical geographical location around the world where AWS clusters data centers.

**Key Characteristics:**

- **Independence:** Each region is completely independent and isolated from other regions
- **Resource Isolation:** Resources in one region don't automatically replicate to others
- **Service Availability:** Not all services are available in all regions
- **Compliance:** Allows data to stay within specific geographic boundaries for regulatory compliance

**Current Global Coverage (as of knowledge cutoff):**
- Over 30 launched regions worldwide
- Examples: us-east-1 (N. Virginia), eu-west-1 (Ireland), ap-south-1 (Mumbai)

**Naming Convention:**
- Geographic area + number (e.g., us-east-1, eu-west-2)
- First region in an area is typically numbered '1'

**Factors for Choosing a Region:**

1. **Latency:** Choose regions close to your end users for better performance
2. **Cost:** Pricing varies between regions based on local infrastructure costs
3. **Compliance:** Legal requirements may mandate data storage in specific countries
4. **Service Availability:** Newer services often launch in certain regions first
5. **Disaster Recovery:** Deploy across multiple regions for high availability

### Availability Zones (AZs)

An **Availability Zone** is one or more discrete data centers within a region, each with redundant power, networking, and connectivity.

**Key Characteristics:**

- **Physical Separation:** Each AZ is physically separated by meaningful distances (often kilometers apart) to protect against disasters
- **Low Latency:** AZs within a region are connected via high-bandwidth, low-latency private fiber networks
- **Independent Infrastructure:** Each AZ has independent power, cooling, and physical security
- **Fault Isolation:** Designed so failures in one AZ don't cascade to others

**Naming Convention:**
- Region code + letter (e.g., us-east-1a, us-east-1b, us-east-1c)
- The letter suffix is account-specific and may map to different physical locations for different accounts

**Number of AZs:**
- Each region has a minimum of 3 AZs (newer regions)
- Some older regions may have 2 AZs
- Maximum currently is 6 AZs per region

**Best Practices:**

1. **Distribute Resources:** Deploy applications across multiple AZs for high availability
2. **Auto Scaling Groups:** Configure to span multiple AZs
3. **Load Balancing:** Use Elastic Load Balancers across AZs
4. **Database Replication:** Enable Multi-AZ deployments for RDS
5. **Data Redundancy:** S3 automatically replicates across AZs within a region

**Use Cases:**

- **High Availability:** If one AZ fails, your application continues running in another
- **Disaster Recovery:** Protect against data center failures
- **Load Distribution:** Balance traffic across multiple AZs
- **Compliance:** Meet requirements for data redundancy

### Edge Locations

**Edge Locations** are AWS data centers designed to deliver content to end users with low latency, primarily used by CloudFront (CDN) and Route 53.

**Key Characteristics:**

- **Global Distribution:** Over 400 edge locations worldwide (more than regions)
- **Content Caching:** Store cached copies of content closer to users
- **Not Full Data Centers:** Smaller facilities focused on content delivery
- **Points of Presence (PoPs):** Edge locations are part of AWS's global network of PoPs

**Services Using Edge Locations:**

1. **CloudFront:** AWS's Content Delivery Network
   - Caches static and dynamic content
   - Reduces latency for global users
   - Supports streaming media

2. **Route 53:** DNS service
   - Provides fast DNS query responses
   - Global availability through edge locations

3. **AWS Global Accelerator:** Improves application availability and performance
   - Uses edge locations as entry points
   - Routes traffic over AWS's private network

4. **AWS WAF (Web Application Firewall):** Deployed at edge locations
   - Filters malicious traffic before it reaches your resources

5. **Lambda@Edge:** Run Lambda functions at edge locations
   - Customize content delivery
   - Execute code closer to users

**Regional Edge Caches:**
- Intermediate caching layer between CloudFront origin servers and edge locations
- Located in regions with high traffic volumes
- Larger cache capacity than standard edge locations
- Help reduce load on origin servers

**Benefits:**

- **Reduced Latency:** Content is served from the nearest edge location to the user
- **Improved Performance:** Faster load times for websites and applications
- **Lower Bandwidth Costs:** Reduces data transfer from origin servers
- **DDoS Protection:** Edge locations can absorb and mitigate attacks
- **Global Reach:** Serve users worldwide without deploying infrastructure everywhere

**Comparison Table:**

| Feature | Region | Availability Zone | Edge Location |
|---------|--------|-------------------|---------------|
| Number | 30+ | 3-6 per region | 400+ |
| Purpose | Deploy resources | High availability | Content delivery |
| Services | All AWS services | Resources span AZs | CDN, DNS, WAF |
| Scope | Geographic area | Data center cluster | Global PoP |
| Connectivity | Independent | Low-latency interconnect | Public internet + AWS backbone |

## Summary

AWS provides a robust, globally distributed cloud infrastructure that enables businesses to deploy applications with high availability, low latency, and compliance with regulatory requirements. Understanding the hierarchy of regions, availability zones, and edge locations is fundamental to architecting effective cloud solutions.

**Key Takeaways:**
- Start with the AWS Free Tier to learn without cost
- Choose regions based on latency, compliance, and cost
- Deploy across multiple AZs for high availability
- Leverage edge locations for global content delivery
- Understand partition differences for specialized requirements (China, GovCloud)
- Core services span compute, storage, database, networking, and security
- 

# AWS Regions List

## AWS Standard Partition (Global Commercial)

### North America
- **us-east-1** - US East (N. Virginia)
- **us-east-2** - US East (Ohio)
- **us-west-1** - US West (N. California)
- **us-west-2** - US West (Oregon)
- **ca-central-1** - Canada (Central)
- **ca-west-1** - Canada West (Calgary)

### South America
- **sa-east-1** - South America (São Paulo)

### Europe
- **eu-west-1** - Europe (Ireland)
- **eu-west-2** - Europe (London)
- **eu-west-3** - Europe (Paris)
- **eu-central-1** - Europe (Frankfurt)
- **eu-central-2** - Europe (Zurich)
- **eu-north-1** - Europe (Stockholm)
- **eu-south-1** - Europe (Milan)
- **eu-south-2** - Europe (Spain)

### Asia Pacific
- **ap-south-1** - Asia Pacific (Mumbai)
- **ap-south-2** - Asia Pacific (Hyderabad)
- **ap-northeast-1** - Asia Pacific (Tokyo)
- **ap-northeast-2** - Asia Pacific (Seoul)
- **ap-northeast-3** - Asia Pacific (Osaka)
- **ap-southeast-1** - Asia Pacific (Singapore)
- **ap-southeast-2** - Asia Pacific (Sydney)
- **ap-southeast-3** - Asia Pacific (Jakarta)
- **ap-southeast-4** - Asia Pacific (Melbourne)
- **ap-southeast-5** - Asia Pacific (Malaysia)
- **ap-east-1** - Asia Pacific (Hong Kong)

### Middle East
- **me-south-1** - Middle East (Bahrain)
- **me-central-1** - Middle East (UAE)
- **il-central-1** - Israel (Tel Aviv)

### Africa
- **af-south-1** - Africa (Cape Town)

## AWS GovCloud (US) Partition

- **us-gov-west-1** - AWS GovCloud (US-West)
- **us-gov-east-1** - AWS GovCloud (US-East)

## AWS China Partition

- **cn-north-1** - China (Beijing) - operated by Sinnet
- **cn-northwest-1** - China (Ningxia) - operated by NWCD

## Announced/Upcoming Regions

AWS regularly announces new regions. Recent announcements include:
- **Mexico**
- **New Zealand**
- **Thailand**
- **Taiwan**
- Additional regions in various countries

## Quick Reference Summary

**Total Regions (approximate as of early 2025):**
- **Standard Partition:** 33+ regions
- **GovCloud Partition:** 2 regions
- **China Partition:** 2 regions

## Notes

- Region availability and service offerings may vary
- New regions are added regularly by AWS
- Some regions require opt-in through the AWS Management Console
- Not all services are available in all regions immediately upon launch
- Pricing varies by region

**To check the most current list of regions**, you can:
1. Visit the [AWS Regional Services List](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)
2. Use AWS CLI: `aws ec2 describe-regions`
3. Check the AWS Management Console under your account's region selector


# Connecting Your Local Machine to AWS (installing aws cli)

## Prerequisites

Before starting, ensure you have:
- An active AWS account
- Administrator access to your local machine
- Basic command line knowledge
- Active internet connection

## Step 1: Create AWS Access Keys

You need programmatic access credentials (Access Key ID and Secret Access Key) to connect from your local machine.

**Steps:**

1. **Log in to AWS Management Console**
   - Go to https://console.aws.amazon.com
   - Sign in with your credentials

2. **Navigate to profile**
   - Go select the top right corner profile name
   - click on Security Credentials
   - Navigate to Access Keys

4. **Create Access Keys**
   - Click **Create access key**

6. **Add Description (Optional)**
   - Add a description tag like "My Local Machine CLI"
   - Click **Create access key**
   - You will get a access key and secret access key

7. **Download and Save Credentials**
   - **IMPORTANT:** This is the only time you can view the Secret Access Key
   - Click **Download .csv file** and save it securely
   - Or copy both Access Key ID and Secret Access Key to a secure location
   - Click **Done**

**Security Warning:** Never share these credentials or commit them to version control (Git). Treat them like passwords.

## Step 2: Download and Install AWS CLI

### For Windows

**Method 1: Using MSI Installer (Recommended)**

1. **Download the AWS CLI Installer**
   - Visit: https://aws.amazon.com/cli/
   - Download the Windows 64-bit installer (MSI)
   - Or direct link: https://awscli.amazonaws.com/AWSCLIV2.msi

2. **Run the Installer**
   - Double-click the downloaded MSI file
   - Follow the installation wizard
   - Accept the license agreement
   - Click **Install**

3. **Verify Installation**
   - Open **Command Prompt** or **PowerShell**
   - Type: `aws --version`
   - You should see output like: `aws-cli/2.x.x Python/3.x.x Windows/10`

**Method 2: Using Chocolatey**

```powershell
choco install awscli
```

### For macOS

**Method 1: Using PKG Installer (Recommended)**

1. **Download the Installer**
   - Visit: https://aws.amazon.com/cli/
   - Download the macOS PKG installer
   - Or direct link: https://awscli.amazonaws.com/AWSCLIV2.pkg

2. **Install**
   - Double-click the downloaded PKG file
   - Follow the installation instructions
   - Enter your password when prompted

3. **Verify Installation**
   - Open **Terminal**
   - Type: `aws --version`

**Method 2: Using Homebrew**

```bash
brew install awscli
```

### For Linux

**Method 1: Using Official Installation Script (Recommended for all distributions)**

```bash
# Download the installer
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Unzip the installer
unzip awscliv2.zip

# Run the install program
sudo ./aws/install

# Verify installation
aws --version
```

**Method 2: Using Package Managers**

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install awscli
```

**CentOS/RHEL/Amazon Linux:**
```bash
sudo yum install awscli
```

**Fedora:**
```bash
sudo dnf install awscli
```

## Step 3: Configure AWS CLI

After installing AWS CLI, you need to configure it with your credentials.

### Basic Configuration

1. **Open Terminal/Command Prompt**

2. **Run Configuration Command**
   ```bash
   aws configure
   ```

3. **Enter Your Credentials**
   
   You'll be prompted for four pieces of information:

   ```
   AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
   AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
   Default region name [None]: us-east-1
   Default output format [None]: json
   ```

   - **AWS Access Key ID:** Paste your Access Key ID from Step 1
   - **AWS Secret Access Key:** Paste your Secret Access Key from Step 1
   - **Default region name:** Choose your preferred region (e.g., us-east-1, eu-west-1, ap-south-1)
   - **Default output format:** Choose from: json, yaml, text, or table (json is recommended)

4. **Verify Configuration**
   ```bash
   aws sts get-caller-identity
   ```

   If successful, you'll see output similar to:
   ```json
   {
       "UserId": "AIDAI234567890EXAMPLE",
       "Account": "123456789012",
       "Arn": "arn:aws:iam::123456789012:user/your-username"
   }
   ```

### Advanced Configuration: Using Named Profiles

If you work with multiple AWS accounts, you can set up named profiles.

**Create Additional Profiles:**

```bash
aws configure --profile profile-name
```

Example:
```bash
aws configure --profile work
aws configure --profile personal
```

**Using Named Profiles:**

```bash
# Use specific profile
aws s3 ls --profile work

# Set default profile for session
export AWS_PROFILE=work

# Or on Windows
set AWS_PROFILE=work
```

**View All Profiles:**

```bash
cat ~/.aws/credentials    # Linux/macOS
type %USERPROFILE%\.aws\credentials    # Windows
```

## Configuration Files Location

AWS CLI stores configuration in two files:

**Credentials File:** `~/.aws/credentials` (Linux/macOS) or `%USERPROFILE%\.aws\credentials` (Windows)
```
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY

[work]
aws_access_key_id = ANOTHER_ACCESS_KEY
aws_secret_access_key = ANOTHER_SECRET_KEY
```

**Config File:** `~/.aws/config` (Linux/macOS) or `%USERPROFILE%\.aws\config` (Windows)
```
[default]
region = us-east-1
output = json

[profile work]
region = eu-west-1
output = table
```


## Next Steps

Now that your local machine is connected to AWS, you can:

1. **Explore AWS Services** via CLI
2. **Write automation scripts** using AWS CLI commands
3. **Use Infrastructure as Code** tools like Terraform or CloudFormation
4. **Install AWS SDKs** for your programming language (Python Boto3, JavaScript SDK, etc.)
5. **Set up AWS CloudShell** for browser-based CLI access
