## Lab: Create MySQL RDS and Connect from EC2 Instance

This lab explains how to create an **Amazon RDS MySQL database** and connect to it from another **EC2 instance** using the **RDS endpoint** and **MariaDB client (mariadb105)**.

---

# Architecture

* **RDS Instance** → MySQL Database
* **EC2 Instance** → Client Machine
* Connection via **RDS Endpoint**

---

# Prerequisites

* AWS Account
* One EC2 Linux instance running
* SSH access to EC2
* Same VPC preferred

---

# Step 1: Create Security Groups

## RDS Security Group

Create security group:

**Name:** `rds-sg`

Inbound Rule:

| Type         | Port | Source             |
| ------------ | ---- | ------------------ |
| MySQL/Aurora | 3306 | EC2 Security Group |

---

## EC2 Security Group

Ensure EC2 has outbound access.

---

# Step 2: Create RDS MySQL Database

Go to:

**AWS Console → RDS → Databases → Create Database**

## Select:

* Standard Create
* Engine Type: **MySQL**
* Version: Latest stable

## Templates:

* Free tier (or Dev/Test)

## Settings:

* DB Instance Identifier: `mydb`
* Master Username: `admin`
* Password: `Password123!`

## Instance Configuration:

* db.t3.micro

## Connectivity:

* VPC: Default or your VPC
* Public Access: No (recommended)
* Security Group: `rds-sg`

## Additional Configuration:

* Initial DB Name: `companydb`

Click **Create Database**

---

# Step 3: Wait Until Available

Status should become:

```bash id="gxzlnh"
Available
```

---

# Step 4: Copy Endpoint

Open DB details.

Example endpoint:

```bash id="gvrh9x"
mydb.xxxxxx.ap-south-1.rds.amazonaws.com
```

Copy this.

---

# Step 5: Connect to EC2

SSH into EC2:

```bash id="h74v3h"
ssh -i key.pem ec2-user@public-ip
```

---

# Step 6: Install MariaDB Client

For Amazon Linux / RHEL:

```bash id="bd07po"
sudo yum install mariadb105 -y
```

Verify:

```bash id="j46xq4"
mysql --version
```

---

# Step 7: Connect to RDS from EC2

Use endpoint:

```bash id="gr0yy0"
mysql -h mydb.xxxxxx.ap-south-1.rds.amazonaws.com -u admin -p
```

Enter password:

```bash id="6nlt7j"
Password123!
```

---

# Step 8: Verify Database Connection

Inside MySQL shell:

```sql id="e2t9cf"
SHOW DATABASES;
```

```sql id="hnqof4"
USE companydb;
```

```sql id="v5zz5f"
CREATE TABLE employees (
id INT PRIMARY KEY AUTO_INCREMENT,
name VARCHAR(50)
);
```

```sql id="x3n83z"
INSERT INTO employees(name) VALUES ('Bhuvan');
```

```sql id="el3tt7"
SELECT * FROM employees;
```

---

# Expected Output

```bash id="lg2gb9"
+----+--------+
| id | name   |
+----+--------+
| 1  | Bhuvan |
+----+--------+
```

---

# Troubleshooting

## Connection Timeout

Check:

* RDS Security Group allows port 3306 from EC2 SG
* EC2 and RDS same VPC/subnet reachable

## Access Denied

Wrong username/password.

## Command Not Found

Install:

```bash id="7yn5e6"
sudo yum install mariadb105 -y
```

---

# Important Notes

* Use **endpoint**, not IP address
* RDS IP may change during failover
* Endpoint remains same

---

# Cleanup

Delete resources after lab:

* RDS Instance
* EC2 Instance (optional)

----
