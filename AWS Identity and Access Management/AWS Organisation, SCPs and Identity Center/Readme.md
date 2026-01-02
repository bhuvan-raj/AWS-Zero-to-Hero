
## 1. AWS Organizations
<img src="https://github.com/bhuvan-raj/AWS-Zero-to-Hero/blob/main/assets/oa.png" alt="Banner" />


### 1.1 What is AWS Organizations?

AWS Organizations is a **centralized account management and governance service** that allows you to:

* Manage **multiple AWS accounts** centrally
* Apply **governance controls** consistently
* Consolidate **billing and cost management**
* Enforce **security guardrails** at scale

It is primarily used in **medium to large enterprises** following a **multi-account strategy**.

---

### 1.2 Why Multi-Account Strategy?

A single AWS account becomes difficult to manage due to:

* Security risks (blast radius)
* Billing complexity
* Compliance requirements
* Environment isolation needs

**Common account separation models:**

* Production / Non-Production
* Dev / Test / QA / Prod
* Security account
* Logging account
* Shared services account

---

### 1.3 Core Components of AWS Organizations

#### a. Management Account

* The **root account** of the organization
* Can create and manage:

  * Member accounts
  * Organizational Units (OUs)
  * SCPs
* Has **full administrative control**
* Best practice: **Do not deploy workloads here**

#### b. Member Accounts

* Individual AWS accounts under the organization
* Can be:

  * Invited existing accounts
  * Newly created accounts
* Subject to SCPs applied at OU or root level

#### c. Organizational Units (OUs)

* Logical containers for grouping accounts
* SCPs are applied at the OU level
* Inheritance applies (Root → OU → Account)

**Example OU structure:**

```
Root
 ├── Security OU
 ├── Production OU
 ├── Non-Production OU
 └── Sandbox OU
```

---

### 1.4 Consolidated Billing

* One bill for all accounts
* Centralized cost visibility
* Volume discounts (tiered pricing)
* Cost allocation tags across accounts

---

### 1.5 Governance vs Permissions

AWS Organizations **does not grant permissions**.
It only **limits the maximum permissions** an account can have.

This distinction is critical.

---

## 2. Service Control Policies (SCP)
<img src="https://github.com/bhuvan-raj/AWS-Zero-to-Hero/blob/main/assets/scp.png" alt="Banner" />

### 2.1 What is an SCP?

An SCP is a **policy that defines the maximum allowed permissions** for:

* All IAM users
* IAM roles
* Root user
  within an AWS account or OU.

SCPs act as **guardrails**, not permission grants.

---

### 2.2 SCP Key Characteristics

* SCPs **do not grant permissions**
* SCPs only:

  * Allow
  * Deny
* SCPs affect **all identities**, including root
* SCPs are evaluated **before IAM policies**
* SCPs are inherited down the hierarchy

---

### 2.3 SCP Evaluation Logic

Effective permissions =
**SCP ∩ IAM Policy ∩ Permission Boundary**

If any layer denies → **Access Denied**

---

### 2.4 Types of SCPs

#### a. Allow List SCP (Default Deny Model)

* Explicitly define allowed services
* Everything else is denied

**Use case:** Highly regulated environments

#### b. Deny List SCP (Default Allow Model)

* Allow everything except explicitly denied actions

**Use case:** Common enterprise governance

---

### 2.5 Common SCP Use Cases

#### 1. Restrict AWS Regions

* Prevent resource creation outside approved regions

#### 2. Protect Security Resources

* Deny deletion of:

  * CloudTrail
  * GuardDuty
  * IAM roles
  * Config rules

#### 3. Enforce Tagging

* Deny resource creation without mandatory tags

#### 4. Prevent Root User Actions

* Deny root user from sensitive operations

---

### 2.6 SCP vs IAM Policy vs Permission Boundary

| Feature               | IAM Policy | SCP          | Permission Boundary |
| --------------------- | ---------- | ------------ | ------------------- |
| Grants permissions    | Yes        | No           | No                  |
| Restricts permissions | Yes        | Yes          | Yes                 |
| Scope                 | Identity   | Account / OU | Identity            |
| Affects root          | No         | Yes          | No                  |
| Used for delegation   | No         | Yes          | Yes                 |

---

### 2.7 SCP Best Practices

* Start with **Deny-based SCPs**
* Test in **sandbox OU first**
* Avoid locking out administrators
* Keep SCPs **simple and readable**
* Always maintain a **break-glass role**

---

## 3. AWS IAM Identity Center (AWS SSO)
<img src="https://github.com/bhuvan-raj/AWS-Zero-to-Hero/blob/main/assets/ic.png" alt="Banner" />


### 3.1 What is IAM Identity Center?

AWS IAM Identity Center is AWS’s **centralized identity and access management service** for:

* AWS accounts
* AWS services
* Third-party applications

It replaces the legacy AWS SSO service.

---

### 3.2 Why IAM Identity Center?

Traditional IAM users do not scale well in enterprises:

* Credential sprawl
* Manual user management
* No centralized identity lifecycle

IAM Identity Center solves this using:

* Central authentication
* Temporary credentials
* Role-based access

---

### 3.3 Core Components

#### a. Identity Source

* Built-in user directory
* Active Directory
* External IdP (SAML 2.0)

#### b. Permission Sets

* Collection of IAM policies
* Similar to IAM roles
* Assigned to:

  * Users
  * Groups
  * Accounts

#### c. Account Assignments

* Define:

  * Who → Which account → With what permissions

---

### 3.4 Authentication Flow

1. User logs in to Identity Center portal
2. Authentication happens via IdP
3. Temporary credentials are generated
4. User assumes role in target AWS account

No long-term credentials are stored.

---

### 3.5 IAM Identity Center vs IAM Users

| Feature               | IAM Users | IAM Identity Center |
| --------------------- | --------- | ------------------- |
| Centralized access    | No        | Yes                 |
| Temporary credentials | No        | Yes                 |
| Enterprise SSO        | Limited   | Native              |
| Multi-account scaling | Poor      | Excellent           |
| Recommended by AWS    | No        | Yes                 |

---

### 3.6 IAM Identity Center with AWS Organizations

* Identity Center integrates **natively**
* Automatically discovers accounts
* Central permission management
* Ideal for:

  * DevOps teams
  * Platform teams
  * Enterprises

---

### 3.7 Real-World Architecture

```
Corporate IdP (AD / Okta)
        ↓
IAM Identity Center
        ↓
AWS Organizations
        ↓
Multiple AWS Accounts
```

---

## 4. Combined Real-World Use Case

**Enterprise Scenario:**

* AWS Organizations for account structure
* SCPs to restrict regions and protect security services
* IAM Identity Center for employee access
* IAM roles for application workloads

This provides:

* Strong security
* Least privilege
* Centralized governance
* Audit readiness

---

## 5. Interview-Focused Key Takeaways

* SCPs define **maximum permissions**
* IAM policies define **granted permissions**
* Identity Center eliminates IAM user sprawl
* SCPs apply to **root user**
* Identity Center uses **temporary credentials**
* Organizations enable **enterprise-scale governance**
