## 1. What is Single Sign-On (SSO)?
<img src="https://github.com/bhuvan-raj/AWS-Zero-to-Hero/blob/main/assets/sso.png" alt="Banner" />

**Single Sign-On (SSO)** is an authentication mechanism that allows a user to **log in once** and gain access to **multiple independent applications or services** without being required to authenticate again for each one.

### Key Idea

> *One identity, one login, multiple applications.*

SSO centralizes authentication and delegates authorization decisions to the individual applications.

---

## 2. Why SSO is Needed

### Problems Without SSO

* Multiple usernames and passwords per user
* Weak password reuse
* High password reset requests
* Poor user experience
* Difficult access revocation when users leave an organization

### Benefits of SSO

* Improved security (centralized authentication)
* Better user experience
* Reduced password fatigue
* Centralized access control
* Easier compliance and auditing
* Faster onboarding and offboarding

---

## 3. Core Components of SSO

1. **User (Principal)**
   The person attempting to access applications.

2. **Identity Provider (IdP)**
   The central system that authenticates users.
   Examples:

   * Azure AD
   * Okta
   * Google Workspace
   * AWS IAM Identity Center

3. **Service Provider (SP) / Application**
   The application the user wants to access.
   Examples:

   * AWS Console
   * GitHub
   * Jira
   * Salesforce

4. **Trust Relationship**
   A predefined trust between IdP and SP using certificates and metadata.

5. **SSO Protocols**

   * SAML 2.0
   * OpenID Connect (OIDC)
   * OAuth 2.0
   * Kerberos (internal networks)

---

## 4. How SSO Works (Step-by-Step Flow)

Below is a **generic SSO flow** (IdP-initiated or SP-initiated, commonly SAML-based):

### Step 1: User Requests Access

* User tries to access an application (e.g., AWS Console).

### Step 2: Redirect to Identity Provider

* Application redirects the user to the IdP for authentication.

### Step 3: User Authentication

* User authenticates using:

  * Username/password
  * MFA (OTP, authenticator app, hardware key)
  * Biometrics (in some systems)

### Step 4: Token/Assertion Issued

* IdP verifies credentials and issues a **security token**:

  * SAML Assertion (XML)
  * OIDC ID Token (JWT)

### Step 5: Token Sent to Application

* Token is sent back to the application securely.

### Step 6: Access Granted

* Application validates the token.
* User is logged in without entering credentials again.

### Step 7: Access to Other Apps

* When accessing another trusted application, authentication is reused.
* No re-login required.

---

## 5. Common SSO Protocols (Brief)

### SAML 2.0

* XML-based
* Widely used in enterprises
* Common in AWS, Azure AD integrations

### OpenID Connect (OIDC)

* Modern, JSON/JWT-based
* Built on OAuth 2.0
* Used in cloud-native and SaaS apps

### OAuth 2.0

* Authorization framework
* Used for delegated access (not authentication alone)

---

## 6. What is SSO in AWS?

<img src="https://github.com/bhuvan-raj/AWS-Zero-to-Hero/blob/main/assets/aws-sso.png" alt="Banner" />


In AWS, SSO is implemented using **AWS IAM Identity Center** (formerly called **AWS SSO**).

### Definition

**AWS IAM Identity Center** enables centralized access management for:

* Multiple AWS accounts
* AWS services
* External applications (SaaS)
* Using existing identity providers

---

## 7. AWS IAM Identity Center – Key Concepts

### 1. Identity Source

Defines where user identities come from:

* IAM Identity Center directory (built-in)
* External IdP (Azure AD, Okta, etc.)

### 2. Users and Groups

* Users are authenticated centrally.
* Groups simplify permission management.

### 3. Permission Sets

* Collection of IAM policies.
* Define what a user/group can do in an AWS account.
* Examples:

  * ReadOnlyAccess
  * AdministratorAccess
  * Custom permission sets

### 4. AWS Accounts

* Works across **AWS Organizations**
* One identity can access multiple accounts

---

## 8. AWS SSO Working Flow (Step-by-Step)

### Step 1: User Accesses AWS Portal

* User opens the IAM Identity Center user portal.

### Step 2: Authentication via Identity Provider

* User authenticates using:

  * IAM Identity Center credentials, or
  * External IdP (e.g., Azure AD with MFA)

### Step 3: Authorization Mapping

* IAM Identity Center checks:

  * User group
  * Assigned permission set
  * Target AWS account

### Step 4: Temporary Credentials Issued

* AWS issues **temporary STS credentials**
* No long-term access keys involved

### Step 5: AWS Console or CLI Access

* User accesses:

  * AWS Management Console
  * AWS CLI
  * SDKs

### Step 6: Session Expiry

* Session expires based on configuration.
* User must re-authenticate after expiry.

---

## 9. AWS SSO with External Identity Providers

AWS supports federation with:

* Azure Active Directory
* Okta
* Google Workspace
* Any SAML 2.0 compatible IdP

### Flow with External IdP

1. User logs in to corporate IdP
2. IdP authenticates user
3. IdP sends SAML assertion to AWS
4. IAM Identity Center maps user to permission set
5. Temporary AWS access granted

---

## 10. Security Advantages of AWS SSO

* No IAM users per account
* No long-term access keys
* Centralized access management
* Built-in MFA support
* Easy user deprovisioning
* Strong auditing with CloudTrail

---

## 11. SSO vs IAM Users (AWS Context)

| Feature              | IAM Users | AWS SSO     |
| -------------------- | --------- | ----------- |
| Credential type      | Long-term | Temporary   |
| Central management   | No        | Yes         |
| Multi-account access | Complex   | Native      |
| MFA enforcement      | Per user  | Centralized |
| Best practice        | ❌         | ✅           |

---

## 12. Real-World Use Case

**Enterprise Scenario**

* Organization has 20 AWS accounts
* Uses Azure AD for employees
* IAM Identity Center integrated with Azure AD
* Developers get read/write access to dev accounts
* Admins get limited production access
* One login, all accounts accessible securely

---

## 13. Summary

* SSO allows users to authenticate once and access multiple systems.
* It relies on trust between Identity Providers and Service Providers.
* AWS implements SSO using IAM Identity Center.
* AWS SSO is a best practice for multi-account, enterprise-scale environments.
* It improves security, scalability, and operational efficiency.
