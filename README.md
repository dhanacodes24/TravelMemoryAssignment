# TravelMemoryAssignment

# Travel Memory Application Deployment on AWS EC2 with Load Balancer and Cloudflare

## Project Overview

This document explains the complete deployment process of the **Travel Memory** MERN Stack application on **Amazon EC2**, including:

- Backend deployment using Node.js
- Frontend deployment using React
- Reverse proxy configuration using Nginx
- Scaling the application using multiple EC2 instances
- Load balancing using AWS Application Load Balancer (ALB)
- Domain management using Cloudflare DNS

---

# Prerequisites

Before starting the deployment, ensure you have the following:

### AWS Requirements
- AWS Account with access to:
  - EC2
  - Target Groups
  - Application Load Balancer (ALB)
  - Security Groups

### Domain Requirements
- A custom domain name.
- A Cloudflare account with your domain added.

### Database Requirements
- A running MongoDB Cluster (MongoDB Atlas recommended).
- A database created named:

```text
travelmemory
```

- The cluster should allow incoming connections from your EC2 instances.

### Software Requirements

- Git
- Node.js
- npm
- Nginx
- Ubuntu EC2 Instance

---

# Project Repository

Repository:

```text
https://github.com/UnpredictablePrashant/TravelMemory
```

Clone the repository:

```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
```

---

# Part 1: Setting Up an EC2 Instance and Deploying the Application

---

# 1.1 Launching an EC2 Instance

### Step 1: Open AWS Console

Navigate to:

```text
AWS Console → EC2 → Instances
```

Click:

```text
Launch Instance
```

### Step 2: Configure Instance

Provide the following:

- Name: `(user defined)`
- AMI: Ubuntu Server 22.04 LTS
- Instance Type: t2.micro (Free Tier)
- Key Pair: Create or select an existing key pair.
- Security Group:

Allow:

| Type | Port |
|------|------|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| Custom TCP | 3000 |
| Custom TCP | 3001 |

Finally, click:

```text
Launch Instance
```

## Screenshot

<img width="1462" height="763" alt="image" src="https://github.com/user-attachments/assets/a9247036-9a3f-47b8-aef6-e8e6ad96d171" />



## 1.2 Initial Server Setup

Connect to the EC2 instance:


Update package lists:

```bash
sudo apt update
```

Install Node.js, npm :

```bash
sudo apt install -y nodejs npm
```

Clone the application:

```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
```

Move into the project directory:

```bash
cd TravelMemory
```

---

## Screenshot

<img width="587" height="137" alt="image" src="https://github.com/user-attachments/assets/7ed27283-bd22-4131-8660-f6ebfb83dae4" />
<img width="1000" height="263" alt="image" src="https://github.com/user-attachments/assets/2e033e14-971e-4de2-ad17-7e891a144e4d" />
<img width="971" height="267" alt="image" src="https://github.com/user-attachments/assets/f691df36-ac7f-475f-a5c2-0c1fd65439f6" />

---

# 1.3 Starting the Backend Server

Navigate to backend directory:

```bash
cd backend
```

Create and configure the `.env` file:

```bash
nano .env
```

Add:

```env
MONGO_URI=your_mongodb_connection_string/travelmemory
PORT=3001
```

Example:

```env
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/travelmemory
PORT=3001
```

Save the file.

Install dependencies:

```bash
npm install
```

Start the backend server:

```bash
node index.js
```

If everything is configured correctly, the backend should start successfully and connect to MongoDB.

---

## Screenshot
<img width="706" height="72" alt="image" src="https://github.com/user-attachments/assets/f09e709c-d137-4e4f-aeed-9086598dfe30" />
<img width="1896" height="537" alt="image" src="https://github.com/user-attachments/assets/ae1fa64c-e324-49c8-a35a-48e972045b92" />

---

# 1.4 Starting the Frontend Server

Open another terminal session and connect to the same EC2 instance.

Navigate to frontend directory:

```bash
cd TravelMemory/frontend
```

Create and configure `.env`:

```bash
nano .env
```

Add:

```env
REACT_APP_BACKEND_URL=http://PUBLIC_IP:3001
```

Save the file.

Install dependencies:

```bash
npm install
```

Start the React application:

```bash
npm start
```

By default, React runs on:

```text
http://PUBLIC_IP:3000
```

---

## Screenshot

<img width="572" height="96" alt="image" src="https://github.com/user-attachments/assets/f8da3ddf-19b8-47e6-b3e9-1fd279b1924c" />
<img width="1087" height="138" alt="image" src="https://github.com/user-attachments/assets/ec38ab08-5bad-4036-8687-44e3e66e553f" />
<img width="1067" height="173" alt="image" src="https://github.com/user-attachments/assets/8bb7411d-d2ba-4877-b7fb-b0fab6b4df74" />


---

# 1.5 Installing and Configuring Nginx

Install Nginx:

```bash
sudo apt install nginx
```

### Why Nginx?

Nginx acts as a **Reverse Proxy Server**.

Instead of users directly accessing port:

```text
3000
```

they can simply access:

```text
http://PUBLIC_IP
```

on port:

```text
80
```

and Nginx forwards the request internally.

---

## Create Configuration File

```bash
sudo nano /etc/nginx/sites-available/travelmemory
```

### What does this command do?

It creates a new Nginx configuration file named:

```text
travelmemory
```

inside:

```text
/etc/nginx/sites-available/
```

---

## Add the Following Configuration

```nginx
server {
    listen 80;
    server_name YOUR_DOMAIN_OR_PUBLIC_IP;

    location / {
        proxy_pass http://PUBLIC_IP:3000;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## What does this configuration do?

- Listens on port 80.
- Receives requests from users.
- Forwards the requests to React running on port 3000.
- Preserves client IP information.
- Supports WebSocket connections.

---

## Enable the Configuration

```bash
sudo ln -s /etc/nginx/sites-available/travelmemory /etc/nginx/sites-enabled/
```

### Purpose

Creates a symbolic link so Nginx starts using the new configuration.

---

## Remove Default Configuration

```bash
sudo rm /etc/nginx/sites-enabled/default
```

### Purpose

Removes the default Nginx webpage.

---

## Test Configuration

```bash
sudo nginx -t
```

### Purpose

Checks whether the configuration file has any syntax errors.

Expected output:

```text
syntax is ok
test is successful
```

---

## Restart Nginx

```bash
sudo systemctl restart nginx
```

### Purpose

Applies the new configuration.

---

Now visit:

```text
http://PUBLIC_IP
```

The Travel Memory application should load successfully.

> Ensure MongoDB is properly configured and connected.

---

## Screenshot

<img width="1482" height="393" alt="image" src="https://github.com/user-attachments/assets/1db31067-6709-401b-8ec9-d2be9aeaaae7" />
<img width="1230" height="91" alt="image" src="https://github.com/user-attachments/assets/b9d0221d-a853-40c9-a78c-a86e2eb1103f" />
<img width="1203" height="118" alt="image" src="https://github.com/user-attachments/assets/6452a541-2d02-4860-827e-9393ad727e29" />

---

## Screenshot
<img width="1462" height="1072" alt="Screenshot 2026-07-03 144607" src="https://github.com/user-attachments/assets/84dca146-d18c-4aa5-ab13-06f6cf1b0e5b" />


---

# Creating the Second EC2 Instance

Repeat all the above steps to create another EC2 instance running the same application.

The final architecture should have:

- EC2 Instance 1
- EC2 Instance 2

Both running:

- Frontend
- Backend
- Nginx

---

# Part 2: Creating and Configuring Target Groups & Load Balancer

---

# 2.1 Creating a Target Group

Navigate to:

```text
EC2 → Target Groups
```

Click:

```text
Create Target Group
```

Configuration:

| Setting | Value |
|---------|--------|
| Target Type | Instances |
| Protocol | HTTP |
| Port | 80 |
| VPC | Default VPC |

Click:

```text
Next
```

---

## Register Targets

Select:

- EC2 Instance 1
- EC2 Instance 2

Click:

```text
Include as Pending Below
```

Then click:

```text
Create Target Group
```

---

## Screenshot

<img width="1023" height="790" alt="Screenshot 2026-07-03 150241" src="https://github.com/user-attachments/assets/9665c196-5383-4f19-a1c7-0b977a8e10b0" />
<img width="1030" height="471" alt="Screenshot 2026-07-03 150333" src="https://github.com/user-attachments/assets/921b0d70-e446-4dad-9342-41054ddef865" />
<img width="1217" height="682" alt="Screenshot 2026-07-03 150401" src="https://github.com/user-attachments/assets/0b369002-e235-49c7-ae0f-1a0dae28b6cb" />
<img width="1081" height="451" alt="image" src="https://github.com/user-attachments/assets/62248337-41c9-4a35-a0e1-726ad36d7849" />

---

# 2.2 Creating an Application Load Balancer

Navigate to:

```text
EC2 → Load Balancers
```

Click:

```text
Create Load Balancer
```

Choose:

```text
Application Load Balancer
```

Configure:

| Setting | Value |
|---------|--------|
| Scheme | Internet Facing |
| Listener | HTTP:80 |
| Availability Zones | Select both zones |

---

## Configure Security Groups

Allow:

- HTTP (80)
- HTTPS (443)

---

## Configure Listener

Select the previously created Target Group.

Click:

```text
Create Load Balancer
```

Wait for provisioning.

---

## Copy DNS Name

Example:

```text
travelmemory-alb-123456.us-east-1.elb.amazonaws.com
```

---

## Important Note

Initially, instances inside the Target Group may show:

```text
Unused
```

After the Load Balancer is associated, they should eventually change to:

```text
Healthy
```

---

## Screenshot
<img width="1437" height="602" alt="Screenshot 2026-07-03 150624" src="https://github.com/user-attachments/assets/1bab6b49-672f-4708-b763-6cc1c29435de" />
<img width="1396" height="665" alt="Screenshot 2026-07-03 150654" src="https://github.com/user-attachments/assets/1d2ee6af-2518-41bb-8a58-81a8d3923c6c" />

---


# Part 3: DNS Management with Cloudflare

---

Login to your Cloudflare account.

Select your domain.

Navigate to:

```text
DNS → Records
```

Click:

```text
Add Record
```

---

# Create CNAME Record

| Field | Value |
|-------|--------|
| Type | CNAME |
| Name | www |
| Target | Load Balancer DNS |
| Proxy Status | Proxied |

Example:

```text
www → travelmemory-alb-123456.us-east-1.elb.amazonaws.com
```

Save the record.

---

## Screenshot

<img width="1351" height="700" alt="image" src="https://github.com/user-attachments/assets/0d118e85-9d74-45c9-873d-0f393898be90" />

---

# Verify Deployment

Open:

```text
https://www.yourdomain.com
```

If everything is configured correctly, the Travel Memory application should load successfully.

---

## Screenshot

<img width="1623" height="1077" alt="Screenshot 2026-07-03 151905" src="https://github.com/user-attachments/assets/f9b5bf49-8d73-42cd-a03e-12e0f70d9c86" />
<img width="1913" height="1078" alt="Screenshot 2026-07-03 152023" src="https://github.com/user-attachments/assets/f74b9735-7aa9-4a4e-97e7-d6f03c49893b" />
<img width="1173" height="1055" alt="Screenshot 2026-07-03 152039" src="https://github.com/user-attachments/assets/8ab78c3c-586c-47e4-b96e-4282cd6c5ce8" />
<img width="637" height="360" alt="Screenshot 2026-07-03 152135" src="https://github.com/user-attachments/assets/e934958d-4aa0-4c02-8d48-6b9352d71c3f" />

---

# Final Architecture

```text
User
  ↓
Cloudflare DNS
  ↓
Application Load Balancer
  ↓
Target Group
  ↓
EC2 Instance 1
EC2 Instance 2
  ↓
Nginx Reverse Proxy
  ↓
React Frontend + Node Backend
  ↓
MongoDB Atlas
```

---

# Deliverables

✅ Travel Memory application deployed on AWS EC2

✅ Multiple EC2 instances configured for scalability

✅ Application Load Balancer configured

✅ Custom domain configured using Cloudflare

✅ Complete deployment documentation

✅ Architecture diagram

---

# Conclusion

This deployment demonstrates how a MERN stack application can be deployed in a scalable and production-oriented environment using:

- Amazon EC2
- Nginx Reverse Proxy
- AWS Target Groups
- Application Load Balancer
- Cloudflare DNS
- MongoDB Atlas

The architecture provides:

- High Availability
- Scalability
- Fault Tolerance
- Better Traffic Distribution
- Easy Domain Management
