# Introduction to AWS Identity and Access Management (IAM)

## 1. Authentication and Authorization

### Authentication

Authentication is the process of **verifying the identity** of a user or service.

* Answers the question: **“Who are you?”**
* In AWS, authentication is done using:

  * Username and password (AWS Management Console)
  * Access Key ID and Secret Access Key (CLI / SDK)
  * Temporary credentials (via IAM Roles and STS)

### Authorization

Authorization is the process of **determining what actions an authenticated identity is allowed to perform**.

* Answers the question: **“What are you allowed to do?”**
* Controlled using **IAM Policies**
* Policies define:

  * Allowed or denied actions
  * AWS services and resources
  * Conditions (optional)

---

## 2. What is AWS IAM?

**AWS Identity and Access Management (IAM)** is a **global AWS service** that allows you to **securely manage access to AWS services and resources**.

Using IAM, you can:

* Create and manage AWS users, groups, and roles
* Define fine-grained permissions using policies
* Control access without sharing root credentials

IAM is:

* Free of cost
* Highly secure
* Centralized access control service

---

## 3. Why AWS IAM?

AWS IAM is required to:

* Follow **security best practices**
* Avoid using the **root user** for daily operations
* Implement **least privilege access**
* Manage users and permissions at scale
* Enable secure access for:

  * Humans (developers, admins)
  * Applications
  * AWS services

Without IAM, every user would need full access, which is **unsafe and unmanageable**.

---

## 4. Identities in AWS IAM

IAM provides the following identities:

1. **IAM Users**
2. **IAM User Groups**
3. **IAM Roles**
4. **Federated Identity**

Each identity represents a **principal** that can authenticate and perform actions in AWS.

---

## 5. Difference Between Root User and IAM User

| Feature                         | Root User                      | IAM User                            |
| ------------------------------- | ------------------------------ | ----------------------------------- |
| Created automatically           | Yes                            | No                                  |
| Email-based login               | Yes                            | No                                  |
| Full access to all AWS services | Yes                            | No (permission-based)               |
| Can be restricted by policies   | No                             | Yes                                 |
| Used for daily operations       | ❌ Not recommended              | ✅ Recommended                       |
| Can create IAM users            | Yes                            | Yes (if permitted)                  |
| Best practice usage             | Account setup and billing only | Daily administration and operations |

**Best Practice:**
Use the **root user only once** to create IAM users and then lock it down with MFA.

---

## 6. What is an AWS IAM User?

An **IAM User** is an identity created within an AWS account that represents:

* A person (developer, admin)
* An application
* A service needing long-term credentials

Each IAM user can have:

* Console access (username + password)
* Programmatic access (access keys)
* Permissions assigned via policies

---

## 7. Creating an IAM User and Providing Login Credentials

### A. Creating an IAM User (Console Access)

**Steps:**

1. Login as root or admin user
2. Go to **IAM → Users → Create user**
3. Enter a **username**
4. Select **Provide user access to the AWS Management Console**
5. Set:

   * Auto-generated or custom password
   * Password reset requirement (recommended)
6. Attach permissions:

   * Add to a group (recommended)
   * Or attach policies directly
7. Review and create the user

**Login Credentials Provided:**

* AWS Console URL
* Username
* Password

---

### B. Creating an IAM User (CLI Access)

**Steps:**

1. Create user:

```bash
aws iam create-user --user-name dev-user
```

2. Create access keys:

```bash
aws iam create-access-key --user-name dev-user
```

**Credentials Provided:**

* Access Key ID
* Secret Access Key

These credentials are used for:

* AWS CLI
* SDKs
* Automation tools (Terraform, Ansible, CI/CD)

---

## 8. What is an AWS IAM User Group?

An **IAM User Group** is a **collection of IAM users**.

Purpose:

* Simplifies permission management
* Policies are attached to the group
* All users in the group inherit the permissions

Example:

* `Developers` group → EC2, S3 read access
* `Admins` group → Full administrative access

**Key Point:**
Permissions are **not attached to groups themselves**, but to policies attached to groups.

---

## 9. Attaching Policies to IAM Users and User Groups

### A. Attaching Policies to an IAM User

**Console:**

1. IAM → Users → Select user
2. Permissions tab
3. Add permissions
4. Choose:

   * Attach existing policies
   * Add user to group
   * Create inline policy

**CLI:**

```bash
aws iam attach-user-policy \
--user-name dev-user \
--policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

---

### B. Attaching Policies to an IAM User Group

**Console:**

1. IAM → User groups → Select group
2. Permissions tab
3. Attach policies

**CLI:**

```bash
aws iam attach-group-policy \
--group-name Developers \
--policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
```

**Best Practice:**
Attach permissions to **groups**, not individual users.

---
