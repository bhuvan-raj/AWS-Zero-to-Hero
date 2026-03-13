# AWS EC2 Key Pairs

A **Key Pair** in Amazon EC2 is a security mechanism used to securely connect to an EC2 instance using **SSH (Secure Shell)**.

It consists of two cryptographic keys:

* **Public Key**
* **Private Key**

AWS stores the **public key**, while the **private key** remains with the user. Authentication occurs when the private key matches the public key stored on the EC2 instance.

---

# 1. What is a Key Pair?

A key pair is used for **secure authentication** when connecting to an EC2 instance.

Instead of using passwords, AWS uses **public-key cryptography** for secure access.

Key pairs help ensure that **only authorized users can connect to an instance**.

---

# 2. Components of a Key Pair

A key pair consists of two keys that work together.

| Key Type    | Description                                                |
| ----------- | ---------------------------------------------------------- |
| Public Key  | Stored by AWS and placed inside the EC2 instance           |
| Private Key | Downloaded by the user and used to connect to the instance |

The private key is usually downloaded as a file with extensions such as:

```bash
.pem
.ppk
```

---

# 3. How Key Pair Authentication Works

The authentication process follows these steps:

1. User launches an EC2 instance and selects a key pair.
2. AWS places the **public key** in the instance.
3. When connecting via SSH, the user provides the **private key**.
4. The instance verifies the private key against the stored public key.
5. If the keys match, access is granted.

Example SSH connection:

```bash
ssh -i my-key.pem ec2-user@<EC2-Public-IP>
```

---

# 4. Where the Public Key is Stored

Inside the EC2 instance, the public key is stored in the user's home directory.

Example location:

```bash
~/.ssh/authorized_keys
```

This file contains the **public keys allowed to access the instance**.

---

# 5. Key Pair Types in AWS

AWS supports two main key pair types.

## RSA Keys

RSA is the most widely used key type.

Characteristics:

* Traditional SSH key type
* Compatible with most SSH clients
* Default option in many systems

Example:

```bash
RSA 2048
RSA 4096
```

---

## ED25519 Keys

ED25519 is a newer and more secure algorithm.

Characteristics:

* Faster authentication
* Stronger security
* Smaller key size

However, older systems may not support it.

---

# 6. Key Pair File Formats

Different operating systems and tools use different formats.

| Format | Usage                    |
| ------ | ------------------------ |
| .pem   | Used by Linux and macOS  |
| .ppk   | Used by PuTTY on Windows |

Conversion example using PuTTYgen:

```bash
.pem → .ppk
```

---

# 7. Creating a Key Pair

Key pairs can be created from the AWS Console.

Steps:

1. Go to **EC2 Dashboard**
2. Navigate to **Key Pairs**
3. Click **Create Key Pair**
4. Select key type
5. Download the private key

Example name:

```
devops-key
```

---

# 8. Security Best Practices

Key pairs must be handled securely.

### Protect the Private Key

Never share the private key with others.

### Restrict File Permissions

For Linux systems:

```bash
chmod 400 my-key.pem
```

### Store Keys Securely

Use secure storage such as:

* AWS Secrets Manager
* Secure password managers
* Encrypted storage

---

# 9. What Happens if You Lose the Private Key?

If the private key is lost, you cannot directly SSH into the instance.

Possible recovery options:

* Create a new instance and attach the root volume
* Modify the **authorized_keys** file
* Use **AWS Systems Manager Session Manager**

---

# 10. Using Key Pairs with Different Operating Systems

## Linux Instances

Access is typically done using SSH.

Example:

```bash
ssh -i my-key.pem ec2-user@public-ip
```

---

## Windows Instances

Windows instances use **RDP (Remote Desktop Protocol)**.

The key pair is used to decrypt the administrator password.

Steps:

1. Launch instance
2. Download password
3. Provide private key to decrypt password
4. Connect via RDP

---

# 11. Key Pair vs Password Authentication

| Feature          | Key Pair             | Password               |
| ---------------- | -------------------- | ---------------------- |
| Security         | Very High            | Lower                  |
| Brute Force Risk | Very Low             | High                   |
| Management       | Requires key storage | Simple but less secure |
| AWS Default      | Yes                  | No                     |

AWS disables password login for Linux instances by default to improve security.

---

# 12. Key Pair in DevOps Workflows

Key pairs are used in many automation workflows.

Examples:

* CI/CD pipelines
* Infrastructure automation
* Remote server management
* Deployment scripts

They are often combined with tools such as:

* Terraform
* Ansible
* Jenkins
* GitHub Actions

---

# Summary

| Concept        | Explanation                              |
| -------------- | ---------------------------------------- |
| Key Pair       | Public-private key authentication system |
| Public Key     | Stored by AWS on the instance            |
| Private Key    | Used by the user to connect              |
| Authentication | Matching keys grant access               |
| Security       | Stronger than password-based login       |

---

# Conclusion

Key pairs are a fundamental security mechanism for accessing EC2 instances. By using public-key cryptography, AWS ensures secure and reliable authentication without relying on traditional passwords.


# SSH Clients and Connecting to AWS EC2 Instances

SSH (Secure Shell) is a cryptographic network protocol used to securely access and manage remote servers over a network.

To connect to Linux-based EC2 instances, users typically use **SSH clients**, which allow secure communication between the local machine and the remote server.

---

# 1. What is an SSH Client?

An SSH client is a software application used to establish a **secure remote connection to a server using the SSH protocol**.

It encrypts all communication between the client and the server, ensuring that data such as **commands, credentials, and responses remain secure**.

---

# 2. Common SSH Clients

Different operating systems use different SSH clients.

| SSH Client         | Platform      | Description                             |
| ------------------ | ------------- | --------------------------------------- |
| OpenSSH            | Linux / macOS | Default command-line SSH client         |
| PuTTY              | Windows       | Popular SSH client for Windows          |
| MobaXterm          | Windows       | Advanced SSH client with built-in tools |
| Bitvise SSH Client | Windows       | Secure graphical SSH client             |

---

# 3. SSH Authentication in AWS EC2

When connecting to an EC2 instance, AWS uses **key pair authentication** instead of passwords.

Requirements for connection:

* EC2 instance must be running
* Public IP or Elastic IP of the instance
* Security group allowing **SSH (Port 22)**
* Private key file (.pem or .ppk)

Example SSH command (Linux):

```bash
ssh -i my-key.pem ec2-user@<public-ip>
```

---

# 4. Connecting to EC2 Using MobaXterm

MobaXterm is a powerful SSH client that provides a graphical interface for remote access.

## Step 1: Download MobaXterm

Download from:

https://mobaxterm.mobatek.net/

Install and launch the application.

---

## Step 2: Create a New SSH Session

1. Open **MobaXterm**
2. Click **Session**
3. Select **SSH**

---

## Step 3: Enter Connection Details

Provide the following information:

Host:

```
EC2 Public IP
```

Remote user:

```
ec2-user
```

For Ubuntu instances:

```
ubuntu
```

---

## Step 4: Configure the Private Key

1. Enable **Use private key**
2. Select your downloaded `.pem` key file

Example:

```
my-key.pem
```

---

## Step 5: Connect to the Instance

Click **OK** or **Connect**.

MobaXterm will establish the SSH connection and open a terminal session.

---

# 5. Connecting to EC2 Using PuTTY

PuTTY is one of the most widely used SSH clients for Windows.

However, PuTTY does not directly support `.pem` files, so they must first be converted to `.ppk`.

---

# Step 1: Convert PEM to PPK

Open **PuTTYgen**.

1. Click **Load**
2. Select the `.pem` file
3. Click **Save private key**

This creates a `.ppk` file.

Example:

```
my-key.ppk
```

---

# Step 2: Open PuTTY

Launch the **PuTTY application**.

---

# Step 3: Enter Instance Details

In the **Host Name** field, enter:

```
ec2-user@<EC2-Public-IP>
```

Example:

```
ec2-user@54.210.10.20
```

Port:

```
22
```

Connection type:

```
SSH
```

---

# Step 4: Configure the Private Key

Navigate to:

```
Connection → SSH → Auth
```

Click **Browse** and select the `.ppk` key.

Example:

```
my-key.ppk
```

---

# Step 5: Connect

1. Return to the **Session** tab
2. Click **Open**

PuTTY will initiate the SSH connection.

The first time you connect, you will see a **security alert** confirming the server fingerprint.

Click **Accept**.

---

# 6. Default SSH Usernames for EC2 Instances

Different operating systems use different default users.

| AMI          | Default Username |
| ------------ | ---------------- |
| Amazon Linux | ec2-user         |
| Ubuntu       | ubuntu           |
| CentOS       | centos           |
| Debian       | admin            |
| RHEL         | ec2-user         |

---

# 7. Common SSH Connection Issues

## Permission Denied (Public Key)

Possible causes:

* Wrong username
* Incorrect key file
* Security group blocking SSH

---

## Timeout Error

Possible causes:

* Port 22 not allowed in security group
* Instance not running
* Wrong IP address

---

# 8. Security Best Practices

* Never share private key files
* Restrict key permissions
* Use security groups to limit SSH access
* Consider using **AWS Systems Manager Session Manager** instead of SSH

Example permission setting:

```
chmod 400 my-key.pem
```

---

# Summary

| Tool      | Platform    | Key Format |
| --------- | ----------- | ---------- |
| OpenSSH   | Linux/macOS | .pem       |
| PuTTY     | Windows     | .ppk       |
| MobaXterm | Windows     | .pem       |

---

# Conclusion

SSH clients allow administrators to securely access and manage EC2 instances. Tools such as MobaXterm and PuTTY make it easy to establish encrypted connections using AWS key pairs.
