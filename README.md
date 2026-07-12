<div align="center">

# ✈️ TravelMemory — MERN Stack Deployment on AWS




### Deploying a full-stack MERN application on EC2 with Nginx, Load Balancing & Cloudflare

[![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://react.dev/)
[![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com/atlas)
[![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)](https://nginx.org/)
[![Amazon EC2](https://img.shields.io/badge/Amazon%20EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)](https://aws.amazon.com/ec2/)
[![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?style=for-the-badge&logo=cloudflare&logoColor=white)](https://www.cloudflare.com/)



## 📖 About this project

**TravelMemory** is a MERN stack application . This repository documents how the app was deployed end-to-end on **AWS EC2**, fronted by **Nginx** as a reverse proxy, distributed across **multiple scaled instances** behind a **Load Balancer**, and served through a **custom domain via Cloudflare**.

> 🔗 Original application source: [UnpredictablePrashant/TravelMemory](https://github.com/UnpredictablePrashant/TravelMemory)

<br/>

## 📑 Table of contents

- [Architecture overview](#-architecture-overview)
- [Tech stack](#-tech-stack)
- [Prerequisites](#-prerequisites)
- [Task 1 — Backend configuration](#-task-1--backend-configuration)
- [Task 2 — Connecting frontend & backend](#-task-2--connecting-frontend--backend)
- [Task 3 — Scaling with multiple instances](#-task-3--scaling-with-multiple-instances)
- [Task 4 — Custom domain via Cloudflare](#-task-4--custom-domain-via-cloudflare)
- [Verification & results](#-verification--results)
-
<br/>

  ## Architecture Diagram → 

<div align="center">
  <img width="896" height="795" alt="9F11B91B-759B-4AC9-B090-EB80CF3930A7" src="https://github.com/user-attachments/assets/fb05d0bc-f611-42f1-ad41-fccbdba90e8e" />
</div>

| Layer | Responsibility |
|---|---|
| **Cloudflare** | DNS management, custom domain, proxy/SSL |
| **Load Balancer** | Distributes traffic across scaled EC2 instances |
| **Frontend tier (EC2 × N)** | Serves the React production build via Nginx on port `80` |
| **Backend tier (EC2 × N)** | Node.js/Express API, reverse-proxied by Nginx to port `3000` |
| **MongoDB Atlas** | Managed database used by the backend |

<br/>


## ✅ Prerequisites

- [ ] AWS account with an EC2 key pair
- [ ] A registered domain added to Cloudflare
- [ ] MongoDB Atlas cluster + connection string
- [ ] Node.js & npm installed locally (for testing)
- [ ] Basic familiarity with Linux/SSH and Nginx

<br/>

---

## 🔧 Task 1 — Backend configuration

### 1.1 Created MongoDB : A running MongoDB Cluster (MongoDB Atlas ).


<!-- 📸 Add screenshot: EC2 instance running + security group inbound rules -->
<div align="center">
<img width="836" height="453" alt="Mongo DB" src="https://github.com/user-attachments/assets/448cff1e-8821-470b-b1d7-f6c0768b9b8b" />
</div>


### 1.2 Launch and connect to the EC2 instance

Launch an Ubuntu EC2 instance, open the required security group ports (`22`, `80`, `443`, and `3000` for testing), and connect via SSH.

<!-- 📸 Add screenshot: EC2 instance running + security group inbound rules -->
<div align="center">
<img width="844" height="428" alt="EC2 instance 1" src="https://github.com/user-attachments/assets/94515c9c-375c-4bc6-9de7-df9c160c3405" />
</div>

## Updated inbound Rules

<div align="center">
<img width="841" height="407" alt="inbound rules1" src="https://github.com/user-attachments/assets/cc67a82a-3164-48b6-897c-bdf94e778bb6" />
<img width="845" height="434" alt="inbound rules2" src="https://github.com/user-attachments/assets/406c4434-e001-4e8b-9fae-6cb328776a54" />

</div>

### 1.2 Clone the repository

```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory/backend
```
<img width="841" height="428" alt="Repo download" src="https://github.com/user-attachments/assets/c7ffcd5e-b570-4b40-9b40-a86921b3f08c" />


### 1.3 Install dependencies

```bash
sudo apt update && sudo apt install -y nodejs npm
npm install
```
<img width="840" height="421" alt="Package update" src="https://github.com/user-attachments/assets/59b0342d-6660-4c48-9ed2-2ff423c1107f" />


### 1.4 Configure environment variables

Create a `.env` file inside the `backend` directory:

```env
MONGO_URI=your_mongodb_connection_string
PORT=3000
```

<!-- 📸 Add screenshot: .env file (mask sensitive values) -->
<div align="center">
<img width="841" height="94" alt="env_backend" src="https://github.com/user-attachments/assets/31624841-a425-430f-ba51-c7b16ec241e9" />
<img width="842" height="197" alt="env frontend" src="https://github.com/user-attachments/assets/96083754-ee32-4608-9106-31a183ac5e0c" />

</div>

### 1.5 Run the backend

```bash
node index.js
# or, for production process management:
pm2 start index.js --name travelmemory-backend
```

<!-- 📸 Add screenshot: backend running / pm2 status -->
<div align="center">
<img width="830" height="302" alt="Backend Up" src="https://github.com/user-attachments/assets/90fe694f-dc33-4c13-a30a-52653b104fb8" />

</div>

### 1.6 Configure Nginx as a reverse proxy

```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

<div align="center">
<img width="839" height="36" alt="nginx new file" src="https://github.com/user-attachments/assets/c6a0f85f-5c4f-481b-a638-2a8cbd90e33a" />

</div>

```bash
sudo ln -s /etc/nginx/sites-available/travelmemory /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

<!-- 📸 Add screenshot: nginx -t success + systemctl status nginx -->
<div align="center">
<img width="818" height="58" alt="Nginx status" src="https://github.com/user-attachments/assets/6374b281-6767-477d-839a-5e62d22b0240" />

</div>

<br/>

---

## 🔗 Task 2  — Scaling with multiple instances

### 2.1 Create additional EC2 instances

Launch a second (and further) EC2 instance(s) for both the frontend and backend tiers, repeating the setup from Tasks 1 & 2 on each.

<!-- 📸 Add screenshot: Multiple EC2 instances in the console -->
<div align="center">
<img width="872" height="440" alt="Instance 2" src="https://github.com/user-attachments/assets/36cdd709-bd27-4551-afb8-94f0fd8d5b23" />

</div>

### 3.2 Create target groups

Create separate target groups for the frontend tier (port `80`) and backend tier (port `3000`), and register the corresponding instances.

<!-- 📸 Add screenshot: Target group configuration -->
<div align="center">
<img width="1246" height="694" alt="Untitled 8" src="https://github.com/user-attachments/assets/34a86457-07dd-40d3-aabc-342e5fcd73ca" />
</div>

### 3.3 Create and configure the Load Balancer

Create an **Application Load Balancer**, attach the target groups, and configure listener rules (path-based routing if using a single ALB for both tiers).

<!-- 📸 Add screenshot: ALB configuration + listener rules -->
<div align="center">
<img width="901" height="388" alt="Load_Balacer1" src="https://github.com/user-attachments/assets/45fdd821-b57a-4ab4-9638-e565e350632d" />
<img width="901" height="388" alt="Load_Balacer1" src="https://github.com/user-attachments/assets/ef5af762-53b3-4133-8e07-44f17b9cdbb2" />

</div>

### 3.4 Confirm healthy targets

<!-- 📸 Add screenshot: Target group health check status = healthy -->
<div align="center">
<img width="1246" height="694" alt="Untitled 8" src="https://github.com/user-attachments/assets/79d1efc3-d10f-486e-85ab-bba524342dfb" />

</div>

<br/>

---

## 🌐 Task 4 — Custom domain via Cloudflare

### 4.1 

### 4.2 Create the CNAME record

Point a subdomain (e.g. `api.yourdomain.com`) to the Load Balancer's DNS endpoint:

| Type | Name | Target | Proxy status |
|---|---|---|---|
| CNAME | `api` | `your-alb-dns-name.elb.amazonaws.com` | Proxied |

<!-- 📸 Add screenshot: CNAME record in Cloudflare DNS settings -->
<div align="center">
<img width="892" height="464" alt="Hostinger cname" src="https://github.com/user-attachments/assets/01f05275-e7b1-420b-a641-6581581ffc1a" />

</div>

### 4.3 Create the A record

Point the root/`www` domain to the frontend EC2 instance's public IP:

| Type | Name | Target | Proxy status |
|---|---|---|---|
| A | `@` / `www` | `<frontend-ec2-public-ip>` | Proxied |


## 🧪 Verification & results

<!-- 📸 Add screenshot: Final live application accessible via custom domain -->
<div align="center">
<img width="842" height="406" alt="DNS URL" src="https://github.com/user-attachments/assets/d17d9b27-e61d-4511-a6b0-836c311f157a" />

</div>

- ✅ Backend reachable via Nginx reverse proxy
- ✅ Frontend successfully calling backend APIs
- ✅ Traffic distributed across multiple instances via Load Balancer
- ✅ Application accessible via custom domain with Cloudflare DNS

