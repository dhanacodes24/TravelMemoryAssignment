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

$\Large{\textcolor{#87CEEB}{\textbf{Task 1 — Backend configuration}}}$

$\Large{\textcolor{#87CEEB}{\textbf{Task 2 — Scaling with multiple instances}}}$

$\Large{\textcolor{#87CEEB}{\textbf{Task 3 — Custom domain via Cloudflare}}}$

$\Large{\textcolor{#87CEEB}{\textbf{Verification and  results}}}$


-
<br/>

  ## Architecture Diagram → 

<div align="center">
  <img width="896" height="795" alt="9F11B91B-759B-4AC9-B090-EB80CF3930A7" src="https://github.com/user-attachments/assets/fb05d0bc-f611-42f1-ad41-fccbdba90e8e" />
</div>


---------------------------------------------------------
<div align="center">

$\large{\textcolor{#87CEEB}{\textbf{How the Architecture Works}}}$

</div>

> 🌐 **Cloudflare DNS** routes traffic from a custom domain to AWS.
>
> ⚖️ An **AWS Load Balancer** distributes requests across multiple EC2 instances.
>
> 🖥️ **Frontend servers (React)** handle user requests, while **backend servers (Node.js)** process API calls.
>
> 🗄️ The backend connects to a **MongoDB** database for data storage.
>
> 📈 **Auto-scaling groups** ensure high availability and scalability by adding/removing EC2 instances as needed.

-----------------------------------------------------------

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

<span style="color:skyblue">


$\textcolor{#87CEEB}{\textbf{We need Mongo DB to stores user travel memories, images, and related metadata. Hence created database travelmemory in MongoDB}}$



</span> 

<!-- 📸 Add screenshot: EC2 instance running + security group inbound rules -->
<div align="center">
<img width="836" height="453" alt="Mongo DB" src="https://github.com/user-attachments/assets/448cff1e-8821-470b-b1d7-f6c0768b9b8b" />
</div>


### 1.2 Launch and connect to the EC2 instance


$\textcolor{#87CEEB}{\textbf{Created first EC2 instance to host Travel Memory application.}}$

$\textcolor{#87CEEB}{\textbf{Launched an Ubuntu EC2 instance, opened the required security group ports ( }}$`5000`, `3001`, `80` ,`443`, and `3000` $\textcolor{#87CEEB}{\textbf{ for testing), and connected via SSH   }}$

<!-- 📸 Add screenshot: EC2 instance running + security group inbound rules -->
<div align="center">
<img width="844" height="428" alt="EC2 instance 1" src="https://github.com/user-attachments/assets/94515c9c-375c-4bc6-9de7-df9c160c3405" />
</div>

## Updated inbound Rules

<div align="center">
<img width="841" height="407" alt="inbound rules1" src="https://github.com/user-attachments/assets/cc67a82a-3164-48b6-897c-bdf94e778bb6" />
</div>


  <div align="center">
<img width="845" height="434" alt="inbound rules2" src="https://github.com/user-attachments/assets/406c4434-e001-4e8b-9fae-6cb328776a54" />

</div>

### 1.3 Clone the repository

$\textcolor{#87CEEB}{\textbf{Cloned git repo of Travel memory application  }}$

```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory/backend
```
<img width="841" height="428" alt="Repo download" src="https://github.com/user-attachments/assets/c7ffcd5e-b570-4b40-9b40-a86921b3f08c" />


### 1.4 Install dependencies

$\textcolor{#87CEEB}{\textbf{Updated packages}}$

```bash
sudo apt update && sudo apt install -y nodejs npm
npm install
```
<img width="840" height="421" alt="Package update" src="https://github.com/user-attachments/assets/59b0342d-6660-4c48-9ed2-2ff423c1107f" />


### 1.5 Configure environment variables

$\textcolor{#87CEEB}{\textbf{Created a}}$  `.env` $\textcolor{#87CEEB}{\textbf{file inside the}}$  `backend` $\textcolor{#87CEEB}{\textbf{directory  }}$ 
```env
MONGO_URI=your_mongodb_connection_string
PORT=3001
```

<!-- 📸 Add screenshot: .env file (mask sensitive values) -->
<div align="center">
<img width="841" height="94" alt="env_backend" src="https://github.com/user-attachments/assets/31624841-a425-430f-ba51-c7b16ec241e9" />
</div>


-----
$\textcolor{#87CEEB}{\textbf{Created a}}$  `.env` $\textcolor{#87CEEB}{\textbf{file inside the}}$  `frontend` $\textcolor{#87CEEB}{\textbf{directory  }}$ 

```env
REACT_APP_BACKEND_URL=http://98.94.60.254:3001
```

<div align="center">
<img width="842" height="197" alt="env frontend" src="https://github.com/user-attachments/assets/96083754-ee32-4608-9106-31a183ac5e0c" />

</div>


### 1.5 Configure Nginx as a reverse proxy

$\textcolor{#87CEEB}{\textbf{Installed Nginx to setup reverse proxy }}$

$\textcolor{#87CEEB}{\textbf{Added file travelmemeory and updated configuration as below , pointed it to new file }}$ 

```bash
sudo ln -s /etc/nginx/sites-available/travelmemory /etc/nginx/sites-enabled/

```

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


-------
<img width="499" height="296" alt="nginx config file" src="https://github.com/user-attachments/assets/d89e4a51-3273-4c2a-9946-113c05a45d12" />


$\textcolor{#87CEEB}{\textbf{ Removed default file  }}$


<div align="center">
<img width="839" height="36" alt="nginx new file" src="https://github.com/user-attachments/assets/c6a0f85f-5c4f-481b-a638-2a8cbd90e33a" />

</div>

$\textcolor{#87CEEB}{\textbf{ Confirmed Nginx status  }}$


```bash


sudo nginx -t

sudo systemctl restart nginx

```

<!-- 📸 Add screenshot: nginx -t success + systemctl status nginx -->
<div align="center">
<img width="818" height="58" alt="Nginx status" src="https://github.com/user-attachments/assets/6374b281-6767-477d-839a-5e62d22b0240" />

</div>

<br/>

---


### 1.7 Run the backend

$\textcolor{#87CEEB}{\textbf{ Started backend with help of below commands and confirmed that backend is up 👍   }}$

```bash
cd backend
npm install
node index.js
```

<!-- 📸 Add screenshot: backend running / pm2 status -->
<div align="center">
<img width="830" height="302" alt="Backend Up" src="https://github.com/user-attachments/assets/90fe694f-dc33-4c13-a30a-52653b104fb8" />

</div>

### 1.7 Run the Frontend

$\textcolor{#87CEEB}{\textbf{ Started frontend with help of below commands and confirmed that frontend is up 👍   }}$


```bash
cd frontend
npm install
npm start
```

<!-- 📸 Add screenshot: backend running / pm2 status -->
<div align="center">
<img width="833" height="455" alt="Front_end up" src="https://github.com/user-attachments/assets/20686c47-0fd1-4d61-91ce-f79e32fca9e4" />
</div>



## 🔗 Task 2  — Scaling with multiple instances

### 2.1 Create additional EC2 instances

$\textcolor{#87CEEB}{\textbf{ Launched a second  EC2 instance for both the frontend and backend tiers, repeated the same setup as instance 1 }}$

<!-- 📸 Add screenshot: Multiple EC2 instances in the console -->
<div align="center">
<img width="872" height="440" alt="Instance 2" src="https://github.com/user-attachments/assets/36cdd709-bd27-4551-afb8-94f0fd8d5b23" />

</div>

### 2.2 Create target groups

$\textcolor{#87CEEB}{\textbf{ Created target group and registred Instance 1 and Instance 2 under this target group }}$
<!-- 📸 Add screenshot: Target group configuration -->
<div align="center">
<img width="1246" height="694" alt="Untitled 8" src="https://github.com/user-attachments/assets/34a86457-07dd-40d3-aabc-342e5fcd73ca" />
</div>

### 2.3 Create and configure the Load Balancer

$\textcolor{#87CEEB}{\textbf{  Created Load Balancer and attached the target groups created in previous step  }}$

------------------------------------------------------------------------------------------


<!-- 📸 Add screenshot: ALB configuration + listener rules -->
<div align="center">
<img width="901" height="388" alt="Load_Balacer1" src="https://github.com/user-attachments/assets/45fdd821-b57a-4ab4-9638-e565e350632d" />
</div>



<div align="center">
<img width="891" height="479" alt="Load_Balacer2" src="https://github.com/user-attachments/assets/3697cf50-db1d-4d73-b409-802d40150255" />

</div>

### 2.4 Confirm healthy targets

$\textcolor{#87CEEB}{\textbf{ Confirmed that the target groups are healthy  }}$


<!-- 📸 Add screenshot: Target group health check status = healthy -->
<div align="center">
<img width="1246" height="694" alt="Untitled 8" src="https://github.com/user-attachments/assets/79d1efc3-d10f-486e-85ab-bba524342dfb" />

</div>

<br/>

---

## 🌐 Task 3 — Custom domain via Cloudflare

### 3.1 Create the CNAME record :-

$\textcolor{#87CEEB}{\textbf{  Create a CNAME record pointing to the load balancer endpoint }}$
$\textcolor{#87CEEB}{\textbf{  (Please node I have used hostinger instead cloudflare due to quick availability )  }}$


<!-- 📸 Add screenshot: CNAME record in Cloudflare DNS settings -->
<div align="center">
<img width="892" height="464" alt="Hostinger cname" src="https://github.com/user-attachments/assets/01f05275-e7b1-420b-a641-6581581ffc1a" />

</div>



## 🧪 Verification & results

$\textcolor{#87CEEB}{\textbf{ Checked the domain URL the live application was accessible via custom domain  }}$


<!-- 📸 Add screenshot: Final live application accessible via custom domain -->
<div align="center">
<img width="842" height="406" alt="DNS URL" src="https://github.com/user-attachments/assets/d17d9b27-e61d-4511-a6b0-836c311f157a" />

</div>

------------------

## Result --> 

## ✅ Custom Domain Deployment Success

We successfully deployed the **Travel Memory MERN application** on AWS EC2 and connected it with a **custom domain**.  
- 🌐 Domain traffic is routed through DNS records pointing to AWS resources.  
- ⚖️ An AWS Load Balancer distributes requests across multiple EC2 instances.  
- 💻 React frontend and 🔧 Node.js backend communicate seamlessly.  
- 🍃 MongoDB database integrated for persistent storage.  
- 🚀 Application is now accessible using our customized domain name.
