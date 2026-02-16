

# 🔐 NACL vs Security Groups in AWS

Security Groups are stateful, instance-level firewalls, whereas Network ACLs (NACLs) are stateless, subnet-level firewalls.

In an AWS VPC, both Security Groups and Network ACLs control inbound and outbound traffic. However, they operate at different layers of the network stack and serve distinct purposes in a defense-in-depth security model.

---

## 📍 Architectural Placement

![Image](https://cloudviz.io/assets/aws-security-group-vs-nacl/security-groups.png)

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/network-acl.png)


**Traffic Flow Example (Inbound from Internet):**

```
Internet
   ↓
Internet Gateway
   ↓
NACL (Subnet Level)
   ↓
Security Group (Instance Level)
   ↓
EC2 Instance
```

Both layers must allow traffic for it to reach the instance.

---

# 🧠 Core Differences

| Feature         | Security Group        | Network ACL (NACL)             |
| --------------- | --------------------- | ------------------------------ |
| Scope           | Instance-level        | Subnet-level                   |
| Type            | Stateful              | Stateless                      |
| Allow/Deny      | Allow only            | Allow and Deny                 |
| Rule Processing | All rules evaluated   | Processed in rule number order |
| Return Traffic  | Automatically allowed | Must be explicitly allowed     |
| Rule Numbers    | No                    | Yes (1–32766)                  |
| Association     | Multiple per instance | One per subnet                 |

---

# 🔄 Stateful vs Stateless Behavior

## 🟢 Security Group (Stateful)

If you allow inbound SSH (port 22):

* Return traffic is automatically allowed.
* No need to create a separate outbound rule.

The connection state is tracked automatically.

---

## 🔴 NACL (Stateless)

If you allow inbound SSH (port 22):

* You **must explicitly allow outbound ephemeral ports (1024–65535)**.
* Otherwise, return traffic will be blocked.

NACL does not remember connection state.

---

# 🔢 Rule Evaluation Logic

### Security Groups

* No rule numbering
* Evaluates all rules
* If traffic matches any allow rule → permitted
* Implicit deny if no rule matches

### NACLs

* Rules evaluated in ascending order
* First match determines outcome
* Supports explicit DENY rules
* Ends with implicit deny

Example:

| Rule # | Type      | Action |
| ------ | --------- | ------ |
| 100    | Allow SSH | ALLOW  |
| 110    | Deny All  | DENY   |

Traffic stops at the first matching rule.

---

# 🔐 Default Behavior

### Default Security Group

* Allows all outbound traffic
* Allows inbound only from same security group

### Default NACL

* Allows all inbound
* Allows all outbound

⚠ When creating a **custom NACL**, everything is denied unless explicitly allowed.

---

# 🧱 When to Use Security Groups

Use Security Groups for:

* Instance-level access control
* Application-specific port access
* Allowing traffic between instances using SG references
* Day-to-day infrastructure security

Security Groups are the primary and most commonly used firewall in AWS.

---

# 🧱 When to Use NACLs

Use NACLs for:

* Subnet-wide traffic filtering
* Blocking specific IP addresses
* Adding an additional security layer
* Meeting compliance requirements
* Implementing coarse-grained network restrictions

NACLs are typically a secondary control layer.

---

# 🔍 Example Scenario Comparison

## Scenario 1: Allow SSH only from Bastion Host

**Best Practice:**

* Security Group: Allow port 22 from Bastion Security Group
* NACL: Keep general allow rules

---

## Scenario 2: Block a malicious IP range

**Best Practice:**

* NACL: Add explicit DENY rule for that IP range
* Security Group: Cannot explicitly deny

---

# 🌐 Understanding Ephemeral Ports (Important)

When a client connects:

```
Client → EC2
Source Port: 50000
Destination Port: 22
```

Return traffic:

```
Source Port: 22
Destination Port: 50000
```

For NACLs, you must allow outbound ephemeral ports (1024–65535) for return traffic.

Security Groups handle this automatically.

---

# 🛡️ Defense-in-Depth Model

AWS network security layers:

```
Route Table
   ↓
NACL (Subnet Firewall)
   ↓
Security Group (Instance Firewall)
   ↓
Operating System Firewall
```

Each layer provides additional filtering and protection.

---

# 🚨 Common Misconfigurations

* Forgetting to allow ephemeral ports in NACL
* Creating a custom NACL without outbound rules
* Assuming NACL is stateful
* Opening 0.0.0.0/0 unnecessarily in Security Groups
* Misordering NACL rule numbers

---

# 📌 Summary

| Security Groups                      | NACLs                          |
| ------------------------------------ | ------------------------------ |
| Stateful                             | Stateless                      |
| Instance-level                       | Subnet-level                   |
| Allow rules only                     | Allow and Deny rules           |
| Easier to manage                     | More granular but complex      |
| Automatically handles return traffic | Requires explicit return rules |

---

## 🎯 Interview-Ready Explanation

> Security Groups act as stateful instance-level firewalls that allow traffic based on defined rules, while NACLs function as stateless subnet-level firewalls that evaluate rules in order and require explicit rules for both inbound and outbound traffic.

---
