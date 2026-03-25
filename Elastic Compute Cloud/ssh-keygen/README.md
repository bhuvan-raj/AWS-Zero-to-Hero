# 🔐 Lab: Recover EC2 Access After Losing Private Key

## 🎯 Scenario

* You lost your `.pem` private key
* You **cannot SSH normally**
* You recover access using:

  * EC2 Instance Connect (browser)
  * Generate new key using `ssh-keygen`
  * Replace `authorized_keys`
  * Download new private key to local machine

---

# 🪜 Step 1: Connect via AWS Console

Go to EC2 → Instance → Connect

Use:
👉 **EC2 Instance Connect (browser-based SSH)**

✔ This works even if you lost your key (if IAM allows)

---

# 🧑‍💻 Step 2: Generate New SSH Key Pair

Inside EC2:

```
ssh-keygen -t rsa -b 2048
```

Press Enter for:

* Default location (`~/.ssh/id_rsa`)
* No passphrase

---

# 📁 Step 3: Verify Keys

```
ls ~/.ssh
```

You will see:

* `id_rsa` → Private key
* `id_rsa.pub` → Public key

---

# 🔁 Step 4: Replace authorized_keys

⚠️ This is the critical step

```
cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys
```

👉 This replaces old key with new one

Set permissions:

```bash id="1p1qf4"
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

---

# 📥 Step 5: Copy Private Key to Local Machine

```
cat id_rsa
```

👉 Copy entire output

---

## On Your Local Machine

Create file - mykey.pem

Paste the key → save


# 🔐 Step 6: Test SSH Access

From local machine:

```bash id="4ntgcl"
ssh -i my-key.pem ec2-user@<EC2-PUBLIC-IP>
```

✔ You should login successfully

---

# 🚫 Common Mistakes

### ❌ Permission denied

Fix:

```
chmod 400 mykey.pem
```

---

### ❌ Wrong authorized_keys format

Ensure:

```bash id="x1n2bo"
cat ~/.ssh/authorized_keys
```

Contains only **one valid public key line**

---

# 🔒 Important Notes

* Never share private key
* Always keep backup
* Avoid deleting all keys unless sure

