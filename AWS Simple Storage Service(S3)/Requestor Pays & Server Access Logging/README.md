# Requester Pays in Amazon S3

## What Is Requester Pays?

**Requester Pays** is an S3 bucket option where the **requester**, instead of the bucket owner, pays for the **data transfer and request costs** associated with accessing objects in the bucket.

By default, the **bucket owner** pays for:

* Storage
* Requests
* Data transfer out

When **Requester Pays** is enabled:

* The **bucket owner** pays only for storage
* The **requester** pays for:

  * Data transfer out
  * GET, PUT, LIST, and other request charges

---

## How Requester Pays Works

* The bucket owner enables **Requester Pays** on the bucket.
* Any requester accessing the bucket must explicitly acknowledge payment responsibility.
* Requests must include the header:

  ```
  x-amz-request-payer: requester
  ```
* If the header is missing, S3 denies the request with **403 Access Denied**.

---

## Use Cases

* Sharing large datasets publicly
* Data lakes accessed by multiple AWS accounts
* Preventing unexpected data transfer costs
* Public research datasets (genomics, weather, analytics)

---

## Limitations and Important Points

* Works only with **authenticated AWS requests**
* **Anonymous access is not allowed**
* Not supported for static website hosting
* Not supported for some cross-region replication scenarios
* Must be explicitly enabled per bucket

---

## Exam-Oriented Key Points (Requester Pays)

* Shifts request and data transfer costs to the requester
* Requires `x-amz-request-payer` header
* Does not allow anonymous access
* Bucket owner still pays storage costs

---

# Server Access Logging in Amazon S3

## What Is Server Access Logging?

**Server Access Logging** provides detailed records of requests made to an S3 bucket. Each log entry contains information about **who accessed what, when, and how**.

Logs are delivered as objects to a **target S3 bucket**.

---

## Information Captured in Logs

Each log record includes:

* Requester identity
* Bucket name
* Object key
* Request time
* Request action (GET, PUT, DELETE)
* HTTP status code
* Error code (if any)
* Bytes sent
* Referrer and user agent

---

## How Server Access Logging Works

* Logging is enabled on a **source bucket**
* Logs are delivered to a **target bucket**
* Log files are written periodically (not real-time)
* Logs are stored as standard S3 objects

---

## Requirements

* Target bucket must be in the **same AWS account**
* Target bucket must grant **log delivery permissions**
* Target bucket must not have logging enabled to itself (to avoid loops)

---

## Use Cases

* Security auditing
* Compliance and governance
* Forensic analysis
* Traffic analysis
* Detecting unauthorized access

---

## Limitations and Considerations

* Logs are delivered with a delay
* No guarantee of log order
* Additional storage and request costs apply
* Not suitable for real-time monitoring
* For advanced logging, CloudTrail data events are preferred

---

## Exam-Oriented Key Points (Server Access Logging)

* Logs are delivered to another S3 bucket
* Provides request-level details
* Not real-time
* Cannot log to the same bucket
* Incurs additional S3 storage costs

---

## Comparison Summary

| Feature         | Requester Pays       | Server Access Logging |
| --------------- | -------------------- | --------------------- |
| Purpose         | Cost control         | Auditing & monitoring |
| Who pays        | Requester            | Bucket owner          |
| Security impact | Prevents cost misuse | Tracks access         |
| Real-time       | No                   | No                    |

---

## Best Practices

* Enable **Requester Pays** for shared or public datasets
* Enable **Server Access Logging** for sensitive or compliance-driven buckets
* Apply lifecycle rules to expire old logs
* Use CloudTrail for API-level auditing



# Lab: Server Access Logging in Amazon S3 (Console Method)

## Objective

To enable **Server Access Logging** on a source S3 bucket and store the access logs in a separate target S3 bucket, then verify log generation.

---

## Architecture Overview

* **Source bucket**: The bucket whose access requests will be logged
* **Target bucket**: The bucket where log files will be stored

> Logs are delivered as objects into the target bucket.

---

## Step 1: Create the Target Bucket (Log Bucket)

1. Open **Amazon S3 → Create bucket**
2. Enter a unique bucket name
   Example: `my-access-log-bucket`
3. Create the bucket

### Important Notes

* The target bucket **must not** enable logging to itself
* The target bucket should be private

---

## Step 2: Create or Select the Source Bucket

1. Create a new bucket
   Example: `my-source-bucket`
   **OR**
2. Select an existing bucket whose access you want to monitor

---

## Step 3: Enable Server Access Logging on the Source Bucket

1. Open the **source bucket**
2. Go to **Properties**
3. Scroll to **Server access logging**
4. Click **Edit**
5. Enable **Server access logging**
6. For **Target bucket**, select:

   * `my-access-log-bucket`
7. Save changes

---

## Step 4: Generate Access Logs

Perform some operations on the source bucket, such as:

* Upload an object
* Download an object
* Delete an object
* List bucket contents

These actions will generate log entries.

---

## Step 6: Verify Log Files in Target Bucket

1. Open the **target bucket**
2. Navigate to the prefix (if configured)
3. You will see log files with names similar to:

```
my-source-bucket-2026-01-22-10-45-23-ABCDEFG
```

> Logs may take **several minutes** to appear.

---

## Step 7: Inspect a Log File

1. Download a log file
2. Open it in a text editor

Each log entry contains:

* Bucket name
* Requester
* Request time
* HTTP operation (GET, PUT, DELETE)
* Object key
* Response status
* Bytes transferred
* User agent

---

## Key Validation Points

* Logs are written only to the **target bucket**
* Logs are **not real-time**
* Logs may arrive out of order
* Each request may generate one or more log records

---

## Exam and Interview Key Points

* Server access logging is **bucket-level**
* Logs are stored as objects in another S3 bucket
* Requires explicit permissions on target bucket
* Not suitable for real-time monitoring
* Additional storage and request costs apply

---









