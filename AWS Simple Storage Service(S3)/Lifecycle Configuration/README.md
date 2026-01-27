

# AWS S3 Lifecycle Configuration – In-Depth Notes

## 1. What is S3 Lifecycle Configuration?

**S3 Lifecycle Configuration** is a set of **rules** that automatically manage objects in an S3 bucket over time.

Using lifecycle rules, you can:

* **Transition objects** between storage classes (for cost optimization)
* **Expire (delete) objects** automatically
* **Delete incomplete multipart uploads**
* **Manage noncurrent object versions** (when versioning is enabled)

👉 The main goal is **cost optimization + data management automation**.

---

## 2. Why Do We Need Lifecycle Rules?

Without lifecycle rules:

* Old data stays forever in expensive storage classes
* Buckets grow uncontrollably
* Manual cleanup is error-prone

With lifecycle rules:

* Data moves automatically to cheaper storage
* Old / unused data is cleaned up
* Compliance and retention policies are easier to enforce

---

## 3. Where Lifecycle Configuration Works

Lifecycle rules apply to:

* **Entire bucket** OR
* **Specific objects**, filtered by:

  * Prefix (e.g., `logs/`)
  * Object tags (e.g., `env=prod`)
  * Size (greater than / less than)

---

## 4. Storage Classes Used in Lifecycle Transitions

Lifecycle rules can move objects between the following storage classes:

### 4.1 Common Storage Classes

| Storage Class                 | Use Case                            |
| ----------------------------- | ----------------------------------- |
| S3 Standard                   | Frequently accessed data            |
| S3 Standard-IA                | Infrequent access                   |
| S3 One Zone-IA                | Infrequent, non-critical data       |
| S3 Intelligent-Tiering        | Unknown access patterns             |
| S3 Glacier Instant Retrieval  | Rare access, milliseconds retrieval |
| S3 Glacier Flexible Retrieval | Archive data                        |
| S3 Glacier Deep Archive       | Long-term archival                  |

⚠️ **You cannot transition objects directly to S3 Standard** (it’s the default).

---

## 5. Lifecycle Rule Components

Each lifecycle rule contains:

### 5.1 Rule Status

* Enabled
* Disabled

### 5.2 Scope (Filter)

You can filter objects by:

* Prefix
* Tags
* Object size

Example:

```
Apply rule to objects with prefix: logs/
```

---

## 6. Transition Actions

### 6.1 Current Version Transitions

Move the **current version** of an object after a certain number of days.

Example:

* Day 0 → S3 Standard
* Day 30 → S3 Standard-IA
* Day 90 → S3 Glacier

📝 Rule:

> “After 30 days, transition to Standard-IA”

---

### 6.2 Noncurrent Version Transitions (Versioning Enabled)

When versioning is enabled:

* Old versions become **noncurrent versions**

You can:

* Transition noncurrent versions to cheaper storage
* Expire noncurrent versions after N days

Example:

* Noncurrent version → Glacier after 30 days
* Delete noncurrent versions after 365 days

---

## 7. Expiration Actions (Deletion)

### 7.1 Object Expiration

Automatically deletes objects after a defined time.

Example:

* Delete objects after 180 days

Common use cases:

* Temporary files
* Logs
* Backup artifacts

---

### 7.2 Delete Marker Expiration (Versioned Buckets)

In versioned buckets:

* Deleting an object adds a **delete marker**
* Lifecycle can remove **expired delete markers**

This keeps the bucket clean and avoids clutter.

---

## 8. Abort Incomplete Multipart Uploads

Multipart uploads that are:

* Started
* But never completed

These consume storage.

Lifecycle rules can:

* Abort incomplete uploads after N days (e.g., 7 days)

This is a **best practice** for all production buckets.

---

## 9. Lifecycle Rules + Versioning

| Feature                    | Non-Versioned Bucket | Versioned Bucket   |
| -------------------------- | -------------------- | ------------------ |
| Current version transition | Yes                  | Yes                |
| Object expiration          | Permanent delete     | Adds delete marker |
| Noncurrent version rules   | ❌ No                 | ✅ Yes              |
| Delete marker cleanup      | ❌                    | ✅ Yes              |

---

## 10. Important Lifecycle Constraints

⚠️ **Key limitations you must remember:**

1. Minimum storage duration charges:

   * Standard-IA / One Zone-IA → **30 days**
   * Glacier → **90 days**
   * Glacier Deep Archive → **180 days**

2. Lifecycle rules run **once per day**, not in real time

3. Transitions happen **asynchronously**

4. Objects smaller than **128 KB** should not be transitioned (cost-inefficient)

---

## 11. Lifecycle Rules and Object Lock

If **Object Lock** is enabled:

* Lifecycle rules **cannot delete objects** under:

  * Compliance mode
  * Governance mode (without bypass permission)
* Retention always overrides lifecycle rules

---

## 12. Common Real-World Use Cases

### Example 1: Log Management

* Store logs in Standard for 30 days
* Move to Glacier after 30 days
* Delete after 365 days

### Example 2: Backup Strategy

* Recent backups → Standard-IA
* Older backups → Glacier Deep Archive

### Example 3: Version Cleanup

* Keep only last 5 versions
* Delete noncurrent versions after 90 days

---

## 13. Lifecycle Rule Example (Conceptual)

```
Rule Name: log-cleanup
Prefix: logs/
Transition:
  Day 30 → Standard-IA
  Day 90 → Glacier
Expiration:
  Day 365 → Delete object
```

---

## 14. Lifecycle vs Manual Management

| Feature           | Lifecycle Rule | Manual |
| ----------------- | -------------- | ------ |
| Cost optimization | Automatic      | Manual |
| Error risk        | Low            | High   |
| Scalability       | Excellent      | Poor   |
| Maintenance       | Minimal        | High   |

---

## 15. Exam & Interview Tips

💡 **Common questions:**

* Can lifecycle delete versioned objects? → Yes (via delete markers)
* Can lifecycle override Object Lock? → No
* Is lifecycle immediate? → No (runs daily)
* Can lifecycle transition to Standard? → No
