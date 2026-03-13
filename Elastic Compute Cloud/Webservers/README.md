# Web Servers – Concepts and Hands-on Lab

A **Web Server** is software that delivers web content (HTML, CSS, JavaScript, images, etc.) to users over the internet using the **HTTP or HTTPS protocol**.

When a user enters a website URL in a browser, the request is sent to a web server, which processes the request and returns the appropriate web page.

Examples of popular web servers include:

* Apache HTTP Server
* Nginx
* Microsoft IIS
* LiteSpeed

---

# 1. How a Web Server Works

The interaction between a browser and a web server follows these steps:

1. A user enters a URL in the browser.
2. The browser sends an **HTTP request** to the server.
3. The server processes the request.
4. The server returns the requested resource.
5. The browser renders the webpage.

Example flow:

```
User Browser
      │
HTTP Request
      │
Web Server
      │
HTML / CSS / JS Response
      │
User Browser displays website
```

---

# 2. Components of a Web Server

| Component           | Description                              |
| ------------------- | ---------------------------------------- |
| HTTP Protocol       | Communication between browser and server |
| Web Server Software | Software handling requests               |
| Static Files        | HTML, CSS, JS, images                    |
| Application Layer   | Backend logic (optional)                 |
| Operating System    | Linux/Windows running the server         |

---

# 3. Types of Web Servers

## Apache HTTP Server

Apache is one of the oldest and most widely used web servers.

Characteristics:

* Open-source
* Highly configurable
* Large community support
* Supports many modules

Common features:

* Virtual hosting
* Authentication modules
* SSL/TLS support

---

## Nginx

Nginx is a modern web server designed for **high performance and scalability**.

Characteristics:

* Event-driven architecture
* High concurrency handling
* Often used as a reverse proxy
* Low memory usage

Common use cases:

* High traffic websites
* Load balancing
* Reverse proxy

---

# 4. Apache vs Nginx

| Feature        | Apache        | Nginx        |
| -------------- | ------------- | ------------ |
| Architecture   | Process-based | Event-driven |
| Performance    | Good          | Very high    |
| Memory usage   | Higher        | Lower        |
| Configuration  | Flexible      | Simpler      |
| Static content | Good          | Excellent    |

---

# 5. Default Web Root Locations

| Server                | Default Web Directory |
| --------------------- | --------------------- |
| Apache (Ubuntu)       | /var/www/html         |
| Apache (Amazon Linux) | /var/www/html         |
| Nginx (Ubuntu)        | /var/www/html         |
| Nginx (Amazon Linux)  | /usr/share/nginx/html |

---

# LAB 1 – Install Nginx on Amazon Linux

## Step 1 – Connect to EC2

```
ssh -i key.pem ec2-user@<public-ip>
```

---

## Step 2 – Update Packages

```
sudo yum update -y
```

---

## Step 3 – Install Nginx

```
sudo amazon-linux-extras install nginx1 -y
```

---

## Step 4 – Start Nginx

```
sudo systemctl start nginx
```

Enable at boot:

```
sudo systemctl enable nginx
```

---

## Step 5 – Verify Installation

Open browser:

```
http://<public-ip>
```

You should see the **Nginx welcome page**.

---

## Step 6 – Host a Website

Create a sample webpage:

```
sudo nano /usr/share/nginx/html/index.html
```

Example content:

```html
<html>
<h1>Welcome to My Nginx Website</h1>
<p>Hosted on Amazon Linux EC2</p>
</html>
```

Restart Nginx:

```
sudo systemctl restart nginx
```

---

# LAB 2 – Install Apache on Amazon Linux

## Step 1 – Install Apache

```
sudo yum install httpd -y
```

---

## Step 2 – Start Apache

```
sudo systemctl start httpd
```

Enable on boot:

```
sudo systemctl enable httpd
```

---

## Step 3 – Host Website

Edit the default page:

```
sudo nano /var/www/html/index.html
```

Example content:

```html
<html>
<h1>Welcome to Apache Server</h1>
<p>Running on Amazon Linux</p>
</html>
```

Restart Apache:

```
sudo systemctl restart httpd
```

---

# LAB 3 – Install Nginx on Ubuntu

## Step 1 – Connect to Instance

```
ssh -i key.pem ubuntu@<public-ip>
```

---

## Step 2 – Update System

```
sudo apt update
```

---

## Step 3 – Install Nginx

```
sudo apt install nginx -y
```

---

## Step 4 – Start Nginx

```
sudo systemctl start nginx
```

Enable at startup:

```
sudo systemctl enable nginx
```

---

## Step 5 – Host Website

Edit webpage:

```
sudo nano /var/www/html/index.html
```

Example:

```html
<html>
<h1>My Ubuntu Nginx Website</h1>
<p>Deployed on EC2</p>
</html>
```

Restart Nginx:

```
sudo systemctl restart nginx
```

---

# LAB 4 – Install Apache on Ubuntu

## Step 1 – Install Apache

```
sudo apt install apache2 -y
```

---

## Step 2 – Start Apache

```
sudo systemctl start apache2
```

Enable on boot:

```
sudo systemctl enable apache2
```

---

## Step 3 – Host Website

Edit webpage:

```
sudo nano /var/www/html/index.html
```

Example:

```html
<html>
<h1>Apache Web Server on Ubuntu</h1>
<p>Website successfully hosted</p>
</html>
```

Restart Apache:

```
sudo systemctl restart apache2
```

---

# 6. Verifying Web Server Status

Check service status:

```
sudo systemctl status nginx
```

or

```
sudo systemctl status httpd
```

---

# 7. Security Group Configuration

Ensure the EC2 **Security Group allows HTTP traffic**.

Required rule:

| Type | Port |
| ---- | ---- |
| HTTP | 80   |

For HTTPS:

| Type  | Port |
| ----- | ---- |
| HTTPS | 443  |

---

# 8. Testing the Website

Open in browser:

```
http://<EC2-Public-IP>
```

Your hosted webpage should appear.

---

# Conclusion

Web servers are essential for delivering content over the internet. Tools such as **Apache and Nginx** allow administrators to host websites, manage HTTP requests, and serve web applications efficiently.

By installing and configuring these servers on EC2 instances, developers can easily deploy and manage web applications in the cloud.
