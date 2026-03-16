# Multi-Server-in-Single-Ec2 using Nginx


## Create a Private IP
1. Go to ENI
2. Select the server attached ENI - Actions - Manage IP address - Add IP address
3. Click on Save ( it will automatically add a secondary private ip for that eni

## Create a Elastic IP

In **Amazon Web Services EC2**:

1. Go to **Elastic IPs**
2. Click **Allocate Elastic IP**
3. Allocate **Elastic IPs**
4. Associate ** IP to the EC2 instance**
5. Select the newly created secondary private IP
   

Example:

```
Elastic IP 1 → 3.110.10.10

```

---

# 2. Verify Multiple IPs on Amazon Linux

SSH into the server and run:

```bash
ip a
```

You should see multiple public IP mappings.

---

# 3. Install Nginx

```bash
sudo dnf install nginx -y
```

Start it:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

# 4. Create Website Directories

Your directory choice is correct.

```bash
sudo mkdir -p /var/www/site1
sudo mkdir -p /var/www/site2
```

Create test pages:

```bash
echo "Site 1" | sudo tee /var/www/site1/index.html
echo "Site 2" | sudo tee /var/www/site2/index.html
```

Set permissions:

```bash
sudo chown -R nginx:nginx /var/www
```

---

# 5. Create Nginx Config for Each IP

Edit config:

```bash
sudo nano /etc/nginx/conf.d/multisite.conf
```

Add:

```nginx
server {
    listen 3.110.10.10:80;  ## change this ip to your private-ip-1
    server_name _;

    root /var/www/site1;
    index index.html;
}

server {
    listen 65.0.20.20:80;  ## change this ip to your private-ip-2
    server_name _;

    root /var/www/site2;
    index index.html;
}
```
### change your Private Ip address accordingly

---

# 6. Test and Restart Nginx

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

# 7. Test in Browser

Open:

```
http://your-private-ip-1
```

You should see:

```
Site 1
```

Open:

```
http://your-private-ip-2
```

You should see:

```
Site 2
```

---

# Important Security Group Rule

Allow **HTTP (port 80)** in your EC2 security group.

---

# Architecture (for your understanding)

```
Internet
   |
   |--- Elastic IP 1 → EC2 → Nginx → /var/www/site1
   |
   |--- Elastic IP 2 → EC2 → Nginx → /var/www/site2
```

