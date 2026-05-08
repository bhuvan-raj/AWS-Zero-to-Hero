# Introduction to AWS RDS

### What is AWS RDS?

**Amazon RDS (Relational Database Service)** is a **managed database service** provided by AWS that makes it easier to set up, operate, scale, and maintain relational databases in the cloud.

Instead of manually installing and managing databases on servers, AWS handles tasks like:

* Hardware provisioning
* OS patching
* Database installation
* Backups
* Monitoring
* Scaling
* High availability

### Simple Definition

**AWS RDS is a service used to run SQL-based databases in the cloud without managing the underlying infrastructure.**

---

## Supported Database Engines

RDS supports multiple relational database engines:

* **MySQL**
* **PostgreSQL**
* **MariaDB**
* **Oracle**
* **Microsoft SQL Server**
* **Amazon Aurora** (AWS optimized database)

---

## Why Use RDS?

### 1. Easy Setup

Create a database in minutes.

### 2. Managed Service

AWS manages maintenance tasks.

### 3. Automated Backups

Daily backups and point-in-time recovery.

### 4. High Availability

Use **Multi-AZ** for automatic failover.

### 5. Scalability

Increase storage or instance size easily.

### 6. Security

Supports:

* IAM authentication
* Encryption
* VPC isolation
* Security Groups

---

## Key Components of RDS

### DB Instance

The actual database server running in AWS.

### Storage

Where database data is stored (SSD / General Purpose / Provisioned IOPS).

### Endpoint

Connection URL used by applications.

Example:

```bash
mydb.xxxxxx.us-east-1.rds.amazonaws.com
```

### Port Numbers

* MySQL → 3306
* PostgreSQL → 5432
* SQL Server → 1433

---

## Multi-AZ Deployment

Creates a **standby replica** in another Availability Zone.

Benefits:

* Automatic failover
* High availability
* Disaster recovery

---

## Read Replicas

Used for read-heavy workloads.

Benefits:

* Offload read traffic
* Improve performance
* Reporting workloads

---

## Backup Options

### Automated Backup

AWS automatically backs up database.

### Manual Snapshot

User creates backup manually.

---

## Security in RDS

* Security Groups control access
* Database username/password
* Encryption at rest (KMS)
* SSL connections

---

## Use Cases

* Web applications
* ERP systems
* Banking apps
* E-commerce websites
* Internal company apps

---

## Example Scenario

A company hosts an e-commerce website.

* App runs on EC2
* Database runs on RDS MySQL
* Multi-AZ enabled
* Read replica for reporting

---

## Advantages Over Self-Managed DB on EC2

| EC2 Database    | RDS             |
| --------------- | --------------- |
| Manual patching | AWS manages     |
| Manual backup   | Automated       |
| Hard scaling    | Easy scaling    |
| Need DBA effort | Less admin work |

---

## Interview One-Line Answer

**Amazon RDS is a fully managed relational database service that simplifies database setup, maintenance, backup, scaling, and high availability in AWS.**

---

## Commands to Connect

### MySQL

```bash
mysql -h endpoint -u admin -p
```

### PostgreSQL

```bash
psql -h endpoint -U admin -d dbname
```

---

## Summary

RDS is best when you need:

* SQL database
* Easy management
* High availability
* Secure cloud database
* Less operational overhead
