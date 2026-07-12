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
<img width="616" height="340" alt="Untitled drawing-2" src="https://github.com/user-attachments/assets/42dd78c9-ffb0-4f6d-9595-30b4dd645e95" />


</div>


### 1.2 Launch and connect to the EC2 instance

Launch an Ubuntu EC2 instance, open the required security group ports (`22`, `80`, `443`, and `3000` for testing), and connect via SSH.

<!-- 📸 Add screenshot: EC2 instance running + security group inbound rules -->
<div align="center">
<img src="./screenshots/ec2-instance-launch.png" alt="EC2 instance launch" width="700"/>
</div>

### 1.2 Clone the repository

```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory/backend
```

### 1.3 Install dependencies

```bash
sudo apt update && sudo apt install -y nodejs npm
npm install
```

<!-- 📸 Add screenshot: npm install output -->
<div align="center">
<img src="./screenshots/backend-npm-install.png" alt="Backend npm install" width="700"/>
</div>

### 1.4 Configure environment variables

Create a `.env` file inside the `backend` directory:

```env
MONGO_URI=your_mongodb_connection_string
PORT=3000
```

<!-- 📸 Add screenshot: .env file (mask sensitive values) -->
<div align="center">
<img src="./screenshots/backend-env-file.png" alt="Backend .env configuration" width="600"/>
</div>

### 1.5 Run the backend

```bash
node index.js
# or, for production process management:
pm2 start index.js --name travelmemory-backend
```

<!-- 📸 Add screenshot: backend running / pm2 status -->
<div align="center">
<img src="./screenshots/backend-running.png" alt="Backend server running" width="700"/>
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

```bash
sudo ln -s /etc/nginx/sites-available/travelmemory /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

<!-- 📸 Add screenshot: nginx -t success + systemctl status nginx -->
<div align="center">
<img src="./screenshots/nginx-reverse-proxy.png" alt="Nginx reverse proxy configuration" width="700"/>
</div>

<br/>

---

## 🔗 Task 2 — Connecting frontend & backend

### 2.1 Clone and configure the frontend

```bash
cd TravelMemory/frontend
npm install
```

### 2.2 Update `urls.js`

Locate `url.js` (or `urls.js`) inside the frontend `src` directory and point it to the backend's public URL / domain:

```js
export const BASE_URL = "http://your-backend-domain-or-ip:3000";
```

<!-- 📸 Add screenshot: urls.js file with updated backend URL -->
<div align="center">
<img src="./screenshots/frontend-urls-config.png" alt="urls.js configuration" width="700"/>
</div>

### 2.3 Build and serve the frontend

```bash
npm run build
sudo cp -r build/* /var/www/html/
```

<!-- 📸 Add screenshot: React build output -->
<div align="center">
<img src="./screenshots/frontend-build.png" alt="Frontend production build" width="700"/>
</div>

### 2.4 Verify end-to-end connectivity

<!-- 📸 Add screenshot: App loaded in browser, network tab showing successful API calls -->
<div align="center">
<img src="./screenshots/frontend-backend-connected.png" alt="Frontend successfully calling backend API" width="800"/>
</div>

<br/>

---

## 📈 Task 3 — Scaling with multiple instances

### 3.1 Create additional EC2 instances

Launch a second (and further) EC2 instance(s) for both the frontend and backend tiers, repeating the setup from Tasks 1 & 2 on each.

<!-- 📸 Add screenshot: Multiple EC2 instances in the console -->
<div align="center">
<img src="./screenshots/ec2-multiple-instances.png" alt="Multiple EC2 instances" width="800"/>
</div>

### 3.2 Create target groups

Create separate target groups for the frontend tier (port `80`) and backend tier (port `3000`), and register the corresponding instances.

<!-- 📸 Add screenshot: Target group configuration -->
<div align="center">
<img src="./screenshots/target-groups.png" alt="Load balancer target groups" width="800"/>
</div>

### 3.3 Create and configure the Load Balancer

Create an **Application Load Balancer**, attach the target groups, and configure listener rules (path-based routing if using a single ALB for both tiers).

<!-- 📸 Add screenshot: ALB configuration + listener rules -->
<div align="center">
<img src="./screenshots/load-balancer-config.png" alt="Application Load Balancer configuration" width="800"/>
</div>

### 3.4 Confirm healthy targets

<!-- 📸 Add screenshot: Target group health check status = healthy -->
<div align="center">
<img src="./screenshots/target-health-check.png" alt="Healthy target group status" width="800"/>
</div>

<br/>

---

## 🌐 Task 4 — Custom domain via Cloudflare

### 4.1 Add the domain to Cloudflare

<!-- 📸 Add screenshot: Domain added to Cloudflare dashboard -->
<div align="center">
<img src="./screenshots/cloudflare-domain-added.png" alt="Domain added to Cloudflare" width="800"/>
</div>

### 4.2 Create the CNAME record

Point a subdomain (e.g. `api.yourdomain.com`) to the Load Balancer's DNS endpoint:

| Type | Name | Target | Proxy status |
|---|---|---|---|
| CNAME | `api` | `your-alb-dns-name.elb.amazonaws.com` | Proxied |

<!-- 📸 Add screenshot: CNAME record in Cloudflare DNS settings -->
<div align="center">
<img src="./screenshots/cloudflare-cname-record.png" alt="Cloudflare CNAME record" width="800"/>
</div>

### 4.3 Create the A record

Point the root/`www` domain to the frontend EC2 instance's public IP:

| Type | Name | Target | Proxy status |
|---|---|---|---|
| A | `@` / `www` | `<frontend-ec2-public-ip>` | Proxied |

<!-- 📸 Add screenshot: A record in Cloudflare DNS settings -->
<div align="center">
<img src="./screenshots/cloudflare-a-record.png" alt="Cloudflare A record" width="800"/>
</div>

### 4.4 Verify SSL/TLS and propagation

<!-- 📸 Add screenshot: SSL/TLS status + site loading over https -->
<div align="center">
<img src="./screenshots/cloudflare-ssl-live.png" alt="Live site over HTTPS via Cloudflare" width="800"/>
</div>

<br/>

---

## 🧪 Verification & results

<!-- 📸 Add screenshot: Final live application accessible via custom domain -->
<div align="center">
<img src="./screenshots/final-app-live.png" alt="Final deployed application" width="850"/>
</div>

- ✅ Backend reachable via Nginx reverse proxy
- ✅ Frontend successfully calling backend APIs
- ✅ Traffic distributed across multiple instances via Load Balancer
- ✅ Application accessible via custom domain with Cloudflare DNS

<br/>

## 🐞 Troubleshooting

<details>
<summary><strong>502 Bad Gateway from Nginx</strong></summary>
<br/>
Usually means the backend (Node.js) process isn't running on the port Nginx is proxying to. Check with <code>pm2 status</code> or <code>sudo systemctl status nginx</code> and confirm the port in your <code>.env</code> matches the <code>proxy_pass</code> value.
</details>

<details>
<summary><strong>Frontend loads but API calls fail</strong></summary>
<br/>
Double-check <code>urls.js</code> points to the correct backend domain/IP, and that the EC2 security group allows inbound traffic on the required port.
</details>

<details>
<summary><strong>Target group shows unhealthy instances</strong></summary>
<br/>
Verify the health check path returns a 200 response, and that the instance's security group allows traffic from the load balancer's security group.
</details>

<details>
<summary><strong>Domain not resolving after adding DNS records</strong></summary>
<br/>
DNS propagation can take a few minutes. Confirm records are set to "Proxied" in Cloudflare, and check nameservers are correctly pointed to Cloudflare at the registrar.
</details>

<br/>

## 🎓 Learnings

- Setting up Nginx as a reverse proxy for a Node.js backend
- Connecting a React frontend to a backend across environments
- Horizontally scaling an application using multiple EC2 instances
- Configuring an AWS Application Load Balancer with target groups
- Managing DNS and custom domains through Cloudflare

<br/>

## 📄 License

This documentation is provided under the [MIT License](LICENSE). The TravelMemory application itself is © its respective author — see the [original repository](https://github.com/UnpredictablePrashant/TravelMemory).

<br/>

<div align="center">

Made with ☕ and a lot of `systemctl restart nginx`

</div>
