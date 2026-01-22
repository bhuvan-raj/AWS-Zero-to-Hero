

# Static Website Hosting in Amazon S3

Amazon S3 provides a built-in **Static Website Hosting** feature that allows you to host static web content such as HTML, CSS, JavaScript, images, and videos without managing any servers.

---

## 1. Difference Between Static Website and Dynamic Website

### Static Website

A static website delivers **pre-built content** exactly as stored on the server.

**Characteristics**:

* Content does not change based on user interaction
* No server-side processing
* Uses only HTML, CSS, JavaScript
* Fast response time
* Highly scalable
* Low cost
* No database required

**Examples**:

* Portfolio websites
* Documentation sites
* Landing pages
* Marketing websites

---

### Dynamic Website

A dynamic website generates content **at runtime** based on user input or backend logic.

**Characteristics**:

* Content changes based on user requests
* Requires server-side languages (Java, Python, PHP, Node.js, etc.)
* Uses databases
* Higher operational complexity
* Requires compute resources

**Examples**:

* E-commerce websites
* Social media platforms
* Banking applications
* Web applications with user authentication

---

### Summary Comparison

| Aspect             | Static Website | Dynamic Website     |
| ------------------ | -------------- | ------------------- |
| Content generation | Pre-built      | Runtime             |
| Server-side logic  | Not required   | Required            |
| Backend/database   | Not required   | Required            |
| Cost               | Very low       | Higher              |
| Scalability        | Built-in       | Needs scaling setup |
| Performance        | Very fast      | Depends on backend  |

---

## 2. Static Website Hosting Feature in Amazon S3

### What Is Static Website Hosting in S3?

Amazon S3 can serve objects directly over HTTP as a website using a **website endpoint**. When enabled, S3 behaves like a web server for static content.

---

### How S3 Static Website Hosting Works

* You upload static files (HTML, CSS, JS) to an S3 bucket
* Enable **Static website hosting**
* Define:

  * **Index document** (for example, `index.html`)
  * **Error document** (for example, `error.html`)
* S3 serves content using a public website endpoint

---

### Website Endpoint Format

```
http://<bucket-name>.s3-website-<region>.amazonaws.com
```

Example:

```
http://my-static-site.s3-website-ap-south-1.amazonaws.com
```

---

### Key Features

* Built-in HTTP server behavior
* Supports index and error pages
* Supports client-side routing (SPA)
* No server management
* Highly durable and scalable
* Pay only for storage and data transfer

---

### Advantages of Static Website Hosting in S3

* **Cost-effective**
  No compute instances or load balancers required

* **Highly scalable**
  Handles millions of requests automatically

* **Highly durable**
  99.999999999% durability

* **Simple architecture**
  No OS, patching, or server maintenance

* **Easy integration**
  Works seamlessly with CloudFront, Route 53, and ACM

* **High performance**
  Can be accelerated globally using CloudFront

---

### Limitations

* Supports **only static content**
* No HTTPS directly on S3 website endpoint
* No server-side processing
* Limited error handling compared to web servers

---

# 3. Lab: Static Website Hosting Using Amazon S3 (Console Method)

### Step 1: Create an S3 Bucket

* Open **Amazon S3 → Create bucket**
* Enter a **globally unique bucket name**
* Choose the region
* Uncheck **Block all public access**
* Acknowledge the warning
* Create the bucket

---

### Step 2: Upload Website File

Prepare the file locally:

**index.html**

```html
<html>
  <head><title>My Static Website</title></head>
  <body>
    <h1>Welcome to My S3 Static Website</h1>
  </body>
</html>
```



Upload the file to the bucket.

---

### Step 3: Enable Static Website Hosting

* Open the bucket
* Go to **Properties**
* Scroll to **Static website hosting**
* Click **Edit**
* Enable **Static website hosting**
* Hosting type: **Host a static website**
* Index document: `index.html`
* Save changes

---

### Step 4: Configure Bucket Policy for Public Access

Go to **Permissions → Bucket policy** and add:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
```

Replace `your-bucket-name` accordingly.

---

### Step 5: Access the Website

Copy the **Bucket website endpoint** from the Static website hosting section and open it in a browser.

You should see your static website.

---

## Best Practices

* Use **CloudFront** for HTTPS and caching
* Restrict bucket access using Origin Access Control (OAC)
* Enable **versioning** for content rollback
* Enable **server access logging**
* Avoid using ACLs

---

## Exam-Oriented Key Points

* S3 static website hosting supports only **static content**
* Requires public read access
* Uses **website endpoint**, not REST endpoint
* Does not support HTTPS directly
* Highly available and scalable
