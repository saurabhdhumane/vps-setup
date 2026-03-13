# 🚀 Production VPS Deployment Guide

This repository provides a **step-by-step guide to deploy a full-stack application on a production VPS** using:

* **Ubuntu Server**
* **Node.js**
* **PM2 Process Manager**
* **Nginx Reverse Proxy**
* **React / Static Frontend**
* **MongoDB Atlas**
* **Let's Encrypt SSL**

---

# 🏗 Architecture

```
Internet
   ↓
Nginx (80 / 443)
   ↓
Frontend (sampleproject.com)
   ↓
Backend API (app.sampleproject.com)
   ↓
Node.js (PM2)
   ↓
MongoDB Atlas
```

---

# 📋 Prerequisites

Before starting you should have:

* A VPS (Ubuntu 22+ / 24+ recommended)
* A domain name
* Backend Git repository
* Frontend Git repository
* MongoDB Atlas connection string

Example server:

```
Ubuntu 24.04
1 vCPU
512MB RAM
10GB Storage
```

---

# 1️⃣ Connect to VPS

```
ssh root@SERVER_IP
```

Example:

```
ssh root@11.111.111.111
```

---

# 2️⃣ Update Server

Always update packages before installation.

```
apt update && apt upgrade -y
```

Install essential tools:

```
apt install curl git ufw -y
```

---

# 3️⃣ Add Swap (Important for Small VPS)

Small servers need swap memory.

```
fallocate -l 1G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

Verify swap:

```
free -m
```

---

# 4️⃣ Install Node.js

```
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs
```

Verify installation:

```
node -v
npm -v
```

---

# 5️⃣ Install PM2

PM2 keeps your Node.js server running even after crashes or reboot.

```
npm install -g pm2
```

Verify:

```
pm2 -v
```

---

# 6️⃣ Setup Backend Application

Create backend directory:

```
mkdir -p /var/www/backend
cd /var/www/backend
```

Clone repository:

```
git clone BACKEND_REPOSITORY_URL .
```

Install dependencies:

```
npm install
```

Create environment variables file:

```
nano .env
```

Example:

```
PORT=3910
NODE_ENV=production
MONGO_URI=YOUR_MONGODB_URI
JWT_SECRET=YOUR_SECRET
```

Start backend using PM2:

```
pm2 start server.js --name backend
```

Save PM2 configuration:

```
pm2 save
pm2 startup
```

Check running processes:

```
pm2 status
```

---

# 7️⃣ Install Nginx

```
apt install nginx -y
```

Start and enable nginx:

```
systemctl start nginx
systemctl enable nginx
```

---

# 8️⃣ Backend Nginx Configuration

Create backend config file:

```
nano /etc/nginx/sites-available/backend
```

```
server {
    listen 80;
    server_name app.sampleproject.com;

    location / {
        proxy_pass http://127.0.0.1:3910;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;

        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable backend configuration:

```
ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled/
```

---

# 9️⃣ Setup Frontend

Create frontend directory:

```
mkdir -p /var/www/frontend
cd /var/www/frontend
```

Clone frontend repository:

```
git clone FRONTEND_REPOSITORY_URL
cd FRONTEND_PROJECT_FOLDER
```

Install dependencies:

```
npm install
```

Build production frontend:

```
npm run build
```

Copy build files:

```
cp -r dist/* /var/www/frontend/
```

---

# 🔟 Frontend Nginx Configuration

```
nano /etc/nginx/sites-available/frontend
```

```
server {
    listen 80;
    server_name sampleproject.com www.sampleproject.com;

    root /var/www/frontend;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Enable frontend site:

```
ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/
```

Remove default nginx config:

```
rm /etc/nginx/sites-enabled/default
```

Test configuration:

```
nginx -t
```

Restart nginx:

```
systemctl restart nginx
```

---

# 1️⃣1️⃣ Configure Firewall

```
ufw allow OpenSSH
ufw allow 'Nginx Full'
ufw enable
```

Check firewall:

```
ufw status
```

---

# 1️⃣2️⃣ Configure DNS

Add DNS records for your domain.

Example:

| Type  | Name | Value             |
| ----- | ---- | ----------------- |
| A     | @    | SERVER_IP         |
| A     | app  | SERVER_IP         |
| CNAME | www  | sampleproject.com |

Example:

```
sampleproject.com → 11.111.111.111
app.sampleproject.com → 11.111.111.111
```

---

# 1️⃣3️⃣ Install SSL (HTTPS)

Install Certbot:

```
snap install core
snap refresh core
snap install --classic certbot
```

Install SSL for frontend:

```
certbot --nginx -d sampleproject.com -d www.sampleproject.com
```

Install SSL for backend:

```
certbot --nginx -d app.sampleproject.com
```

Test automatic renewal:

```
certbot renew --dry-run
```

---

# 🛠 Useful Commands

View backend logs:

```
pm2 logs backend
```

Restart backend:

```
pm2 restart backend
```

Check nginx status:

```
systemctl status nginx
```

Restart nginx:

```
systemctl restart nginx
```

Monitor server resources:

```
htop
```

---

# 🔄 Deployment Update Workflow

### Update Backend

```
cd /var/www/backend
git pull
npm install
pm2 restart backend
```

### Update Frontend

```
cd /var/www/frontend/FRONTEND_PROJECT_FOLDER
git pull
npm install
npm run build
cp -r dist/* /var/www/frontend/
systemctl restart nginx
```

---

# 🌐 Final URLs

Frontend:

```
https://sampleproject.com
```

Backend API:

```
https://app.sampleproject.com
```

---

# 📁 Server Folder Structure

```
/var/www
   ├── backend
   │      server.js
   │      routes
   │      models
   │
   └── frontend
          index.html
          assets
```

---

# ✅ Deployment Complete

Your **production VPS server is now ready**.

You now have:

* Node.js backend running with **PM2**
* React frontend served via **Nginx**
* Domain routing
* **HTTPS SSL**
* Firewall security
* Production-ready setup
