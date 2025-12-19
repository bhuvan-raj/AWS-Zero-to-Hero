# AWS IAM Policy

## What is an IAM Policy?

Key points:

* JSON document
* Controls access to AWS services
* Answers 4 questions:

  * Who?
  * What action?
  * On which resource?
  * Under what condition?


---

## IAM Policy Structure (JSON Anatomy)

Teach each block individually:

| Element   | Purpose                      |
| --------- | ---------------------------- |
| Version   | Policy language version      |
| Statement | One or more permission rules |
| Effect    | Allow or Deny                |
| Action    | AWS API calls                |
| Resource  | AWS ARN                      |
| Condition | Extra constraints            |


---

### 4. Explicit vs Implicit Deny

Teach **evaluation logic early**:

* Default = Implicit Deny
* Explicit Deny always wins
* Why AWS is deny-first



---

## Policy Categories

## Identity-Based Policies

Attach to:

* User
* Group
* Role

### Sub-categories:

### AWS Managed Policies
- AWS Managed Policies are predefined IAM policies created and maintained by AWS.
They provide ready-made permissions for common AWS use cases.
- AWS Managed Policies are pre-built permission templates that you can directly attach to users, groups, or roles.
  
### Customer Managed Policies
- Customer Managed Policies are IAM policies created, owned, and managed by you (the AWS customer).
They define custom permissions tailored to your organization’s exact requirements.
- Customer Managed Policies are custom-built permission policies that you create and reuse across IAM users, groups, and roles.

### Inline Policies
- Inline Policies are IAM policies embedded directly into a single IAM identity (user, group, or role).
- Inline policies are one-to-one policies that belong to only one identity and cannot be reused.
- Deleted automatically when the identity is deleted

Teaching focus:

* Reusability
* Real-world usage
* Best practice (Customer Managed)

📌 Lab: Attach S3 read-only policy to a user.

---

## Resource-Based Policies

Attach to resources like:

* S3
* SQS
* SNS
* Lambda

Key concepts:

* Uses `Principal`
* Supports cross-account access
* Difference from identity-based policies

# Comparison table:
## Identity-based vs Resource-based


| Aspect               | Identity-Based Policies                        | Resource-Based Policies                              |
| -------------------- | ---------------------------------------------- | ---------------------------------------------------- |
| Attached To          | IAM users, groups, roles                       | AWS resources                                        |
| Applied On           | IAM identities                                 | S3 buckets, SQS queues, SNS topics, Lambda functions |
| Defines              | What an identity can do                        | Who can access the resource                          |
| Contains `Principal` | No                                             | Yes                                                  |
| Cross-Account Access | No (directly)                                  | Yes                                                  |
| Common Examples      | AWS managed, customer managed, inline policies | S3 bucket policy, SQS queue policy                   |
| Scope                | Identity-centric                               | Resource-centric                                     |
| Reusability          | High                                           | Limited to the resource                              |
| Policy Location      | IAM service                                    | Resource configuration                               |
| Typical Use Case     | Grant permissions to users or roles            | Share resources across accounts                      |


---

# AWS IAM Roles

## 1. What Is an IAM Role?

An **IAM Role** is an AWS identity that:

* Has **permissions**, but
* Has **no long-term credentials** (no username or password)

In simple words:

> An IAM role is a **temporary identity** that AWS services or users can assume to perform actions.

---

## 2. Why IAM Roles Are Required

IAM roles solve key security problems:

* Avoid sharing access keys
* Enable **temporary, rotating credentials**
* Allow AWS services to access other AWS services securely
* Enable **cross-account access**

✔ Roles are **more secure** than IAM users.

---

## 3. Key Characteristics of IAM Roles

* No permanent credentials
* Uses **temporary credentials** via AWS STS
* Can be assumed by:

  * AWS services
  * IAM users
  * Other AWS accounts
  * External identity providers (SSO, OIDC)
* Permissions are defined using **policies**

---

## 4. Two Policies in Every Role (Very Important)

### 4.1 Trust Policy (Who Can Assume the Role)

* Resource-based policy
* Uses `Principal`
* Allows `sts:AssumeRole`

Example:

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

---

### 4.2 Permission Policy (What the Role Can Do)

* Identity-based policy
* Defines allowed actions and resources

Example:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

📌 **Trust policy = WHO**
📌 **Permission policy = WHAT**

---

## 5. How Role Assumption Works (Flow)

1. An entity requests to assume a role
2. AWS checks the **trust policy**
3. If allowed, AWS STS issues **temporary credentials**
4. The entity uses these credentials to access resources
5. Credentials expire automatically

---

## AWS STS (Security Token Service) – Study Notes
**1. What Is AWS STS?**

AWS STS (Security Token Service) is an AWS service that issues temporary security credentials.

In simple words:

AWS STS provides short-lived credentials to access AWS resources securely.

These credentials include:

- Access Key ID

- Secret Access Key

- Session Token

## 6. Common Types of IAM Roles

### 6.1 Service Roles

Used by AWS services.

Examples:

* EC2 role accessing S3
* Lambda role accessing DynamoDB
* ECS task role pulling images from ECR

---

### 6.2 Cross-Account Roles

Used to access resources in another AWS account.

Example:

* Dev account accessing Prod account S3

---

### 6.3 IAM User Assumable Roles

Used to avoid giving direct permissions to users.

Example:

* Admin role
* Read-only auditor role

---

### 6.4 Federated Roles

Used with:

* AWS SSO
* Active Directory
* Google / Azure AD (OIDC / SAML)

---

## 7. IAM Role vs IAM User

| Aspect        | IAM Role                            | IAM User           |
| ------------- | ----------------------------------- | ------------------ |
| Credentials   | Temporary                           | Long-term          |
| Use Case      | Services, cross-account, automation | Human users        |
| Security      | More secure                         | Less secure        |
| Key Rotation  | Automatic                           | Manual             |
| Best Practice | Preferred                           | Avoid for services |

---

## 8. Role Session Duration

* Default: **1 hour**
* Maximum:

  * 12 hours (for AWS accounts)
  * Depends on trust policy and STS settings

---

## 9. Real-World Use Cases

* EC2 → S3 access without keys
* Jenkins assuming deployment role
* Lambda accessing DynamoDB
* Cross-account read-only access
* Kubernetes IRSA (EKS → IAM role)

---

## 10. Best Practices for IAM Roles

* Always use roles for AWS services
* Apply least-privilege permission policies
* Use permissions boundaries with roles
* Restrict trust policies tightly
* Enable CloudTrail logging
* Avoid wildcard principals

---
