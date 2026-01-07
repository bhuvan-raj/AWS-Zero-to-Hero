# AWS IAM – Policy Evaluation Logic & Cross‑Account Access Labs

This README serves as a **study note + hands‑on lab guide** for understanding how AWS IAM evaluates permissions and how cross‑account access works, both **within the same AWS account** and **between different AWS accounts**.

---

## 1. IAM Policy Evaluation Logic

###  What is Policy Evaluation?

When an IAM principal (user, role, or service) makes a request to AWS, IAM evaluates **all applicable policies** to decide whether the request is **allowed or denied**.

AWS follows a **strict evaluation order** to determine the final decision.

---


###  IAM Policy Evaluation Flow (Important)

AWS evaluates requests using the following logic:

1. **By default, everything is implicitly denied**
2. AWS checks for **explicit DENY** in any policy

   * If found → **Access Denied (final)**
3. AWS checks for **explicit ALLOW**
4. Permission boundaries and SCPs are applied as **filters**
5. If at least one ALLOW exists and no DENY blocks it → **Access Allowed**

> 📌 **Explicit DENY always overrides ALLOW**

---

###  Policy Evaluation Summary Table

| Scenario                                   | Result    |
| ------------------------------------------ | --------- |
| No policy allows the action                | ❌ Denied  |
| Explicit DENY exists                       | ❌ Denied  |
| ALLOW exists, no DENY                      | ✅ Allowed |
| ALLOW exists but blocked by SCP / boundary | ❌ Denied  |

---

# Permission Boundaries in AWS IAM

## 1. What is a Permission Boundary?

A **Permission Boundary** is an **IAM managed policy** that defines the **maximum permissions** an IAM **user or role** can have.

> A permission boundary **does not grant permissions**.
> It only **limits** what the attached IAM policies are allowed to grant.

---

## 2. Why Permission Boundaries Are Needed

Permission boundaries are used to:

* Prevent **privilege escalation**
* Enable **delegated administration**
* Enforce **least privilege**
* Control permissions in **DevOps and automation workflows**

They are especially useful when you allow teams to create roles or users but **do not want them to exceed certain permissions**.

---

## 3. Where Permission Boundaries Apply

* ✅ IAM Users
* ✅ IAM Roles
* ❌ IAM Groups
* ❌ AWS Accounts / OUs
* ❌ Root user

📌 Permission boundaries apply **only at the IAM identity level**.

---

## 4. How Permission Boundaries Work (Evaluation Logic)

An IAM identity can perform an action **only if ALL of the following allow it**:

1. Identity-based policy allows the action
2. Permission boundary allows the action
3. Resource-based policy (if applicable) allows the action

❌ If the permission boundary does **not allow** the action → **final result is DENY**, even if the IAM policy allows it.

---

## 8. Common Use Cases

* Allow teams to create roles **only with limited permissions**
* Prevent developers from modifying IAM
* Restrict CI/CD pipelines to specific services
* Control third-party access
* Enforce security policies without full admin access

---

## 9. Permission Boundary vs IAM Policy

| Aspect           | IAM Policy          | Permission Boundary |
| ---------------- | ------------------- | ------------------- |
| Purpose          | Grants permissions  | Limits permissions  |
| Grants access    | ✅ Yes               | ❌ No                |
| Enforces maximum | ❌ No                | ✅ Yes               |
| Attached to      | User / Role / Group | User / Role         |

---

#  Lab 1 – Cross‑Access Simulation Within the Same AWS Account

> Purpose: Understand **role assumption logic** without involving a second AWS account.

###  Architecture

* IAM User: `dev-user`
* IAM Role: `s3-read-role`
* S3 Bucket: `my-lab-bucket`

User assumes a role **inside the same AWS account**.

---

### Step 1: Create S3 Bucket

* Create a bucket: `my-lab-bucket`

---

###  Step 2: Create IAM Role

**Role Name:** `s3-crossaccess`

**Trusted Entity (Trust Policy):**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<ACCOUNT_ID>:user/dev-user"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

###  Step 3: Attach Permissions to Role

Attach `s3fullaccess` policy to `s3-read-role`:



---

###  Step 4: Allow User to Assume Role

Attach this policy to `dev-user`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::<ACCOUNT_ID>:role/s3-read-role"
    }
  ]
}
```

---

###  Step 5: Test Using AWS Management Console (GUI)

1. Sign in to AWS Console as **dev-user**
2. Open **IAM → Roles → s3-read-role**
3. Click **Switch role**
4. Enter:

   * Account ID: `<ACCOUNT_ID>`
   * Role name: `s3-read-role`
   * Display name: `S3ReadSession`
5. Switch role successfully
6. Navigate to **S3 → my-lab-bucket**
7. Open `test.txt`

✅ File opens successfully using the assumed role

---

## Lab 2 – Cross‑Account Access Using Two AWS Accounts

> Purpose: Understand **real‑world cross‑account access** using trust policies and resource permissions.

---

###  Architecture

* **Account A** (Resource Owner)

  * S3 Bucket: `cross-account-bucket`
  * IAM Role: `cross-account-role`

* **Account B** (Accessing Account)

  * IAM User: `external-user`

---

###  Step 1: Create S3 Bucket in Account A

Bucket Name: `cross-account-bucket`

Upload: `data.txt`

---

###  Step 2: Create IAM Role in Account A

**Role Name:** `cross-account-role`

**Trust Policy (Allows Account B):**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<ACCOUNT_B_ID>:root"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

###  Step 3: Attach Permissions to Role (Account A)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cross-account-bucket/*"
    }
  ]
}
```

---

###  Step 4: Create IAM User in Account B

User Name: `external-user`

Attach policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::<ACCOUNT_A_ID>:role/cross-account-role"
    }
  ]
}
```

---

###  Step 5: Assume Role from Account B Using Console (GUI)

1. Sign in to **Account B** as `external-user`
2. Open **IAM → Roles**
3. Click **Switch role** (top-right corner)
4. Enter:

   * Account ID: `<ACCOUNT_A_ID>`
   * Role name: `cross-account-role`
   * Display name: `CrossAccountAccess`
5. Switch role successfully
6. Navigate to **S3 service**
7. Open bucket `cross-account-bucket`
8. Open `data.txt`

✅ Cross-account access verified via AWS Console

---

## 4. Key Interview & Teaching Takeaways

* IAM always starts with **implicit deny**
* **Explicit deny overrides everything**
* Cross‑account access **always uses IAM Roles**, never direct user access
* Trust policy defines **who can assume** the role
* Permission policy defines **what the role can do**

---

## 5. Common Mistakes to Highlight to Students

* Forgetting `sts:AssumeRole` permission
* Incorrect account ID in trust policy
* Confusing resource policy with identity policy
* Expecting IAM users to access another account directly

---

✅ This README can be directly used for **teaching, labs, and revision**.
