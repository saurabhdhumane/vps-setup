# Production VPS Deployment Guide

## Project: sampleProject 01

This document explains how to deploy the **sampleProject full-stack application** on a production VPS.

Architecture:

```
Internet
   ↓
Nginx (80/443)
   ↓
Frontend (sampleProject.com)
   ↓
Backend API (app.sampleProject.com)
   ↓
Node.js (PM2)
   ↓
MongoDB Atlas
```

---

# 1. Connect to VPS

```
ssh root@SERVER_IP
```

Example:

```
ssh root@64.227.174.172
```

---

# 2. Update Server

```
apt update && apt upgrade -y
```

Install useful tools:

```
apt install curl git ufw -y
```

---

# 3. Add Swap (Important for small VPS)

```
fallocate -l 1G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

Verify:

```
free -m
```

---

# 4. Install Node.js

```
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs
```

Verify:

```
node -v
npm -v
```

---

# 5. Install PM2

PM2 keeps the Node server running.

```
npm install -g pm2
```

Check:

```
pm2 -v
```

---

# 6. Setup Backend

Create project directory:

```
mkdir -p /var/www/backend
cd /var/www/backend
```

Clone backend repo:

```
git clone BACKEND_REPO_URL .
```

Install dependencies:

```
npm install
```

Create `.env` file:

```
nano .env
```

Example:

```
PORT=3910
NODE_ENV=production
MONGO_URI=YOUR_MONGO_URI
JWT_SECRET=YOUR_SECRET
```

Start backend with PM2:

```
pm2 start server.js --name backend
```

Save PM2 config:

```
pm2 save
pm2 startup
```

Check:

```
pm2 status
```

---

# 7. Install Nginx

```
apt install nginx -y
```

Start service:

```
systemctl start nginx
systemctl enable nginx
```

---

# 8. Backend Nginx Configuration

Create config:

```
nano /etc/nginx/sites-available/backend
```

```
server {
    listen 80;
    server_name app.sampleProject.com;

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

Enable it:

```
ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled/
```

---

# 9. Setup Frontend

Create directory:

```
mkdir -p /var/www/frontend
cd /var/www/frontend
```

Clone frontend:

```
git clone FRONTEND_REPO_URL
cd FRONTEND_FOLDER
```

Install dependencies:

```
npm install
```

Build production files:

```
npm run build
```

Copy build output:

```
cp -r dist/* /var/www/frontend/
```

---

# 10. Frontend Nginx Configuration

```
nano /etc/nginx/sites-available/frontend
```

```
server {
    listen 80;
    server_name sampleProject.com www.sampleProject.com;

    root /var/www/frontend;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Enable site:

```
ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/
```

Remove default config:

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

# 11. Configure Firewall

```
ufw allow OpenSSH
ufw allow 'Nginx Full'
ufw enable
```

---

# 12. Configure DNS

DNS Records:

```
A    @      SERVER_IP
A    app    SERVER_IP
CNAME www   sampleProject.com
```

Example:

```
sampleProject.com → 64.227.174.172
app.sampleProject.com → 64.227.174.172
```

---

# 13. Install SSL (HTTPS)

Install Certbot:

```
snap install core
snap refresh core
snap install --classic certbot
```

Frontend SSL:

```
certbot --nginx -d sampleProject.com -d www.sampleProject.com
```

Backend SSL:

```
certbot --nginx -d app.sampleProject.com
```

Test auto-renew:

```
certbot renew --dry-run
```

---

# 14. Useful Commands

Check backend logs:

```
pm2 logs backend
```

Restart backend:

```
pm2 restart backend
```

Check nginx:

```
systemctl status nginx
```

Restart nginx:

```
systemctl restart nginx
```

Check server usage:

```
htop
```

---

# 15. Deployment Update Flow

When updating backend:

```
cd /var/www/backend
git pull
npm install
pm2 restart backend
```

When updating frontend:

```
cd /var/www/frontend/FRONTEND_FOLDER
git pull
npm install
npm run build
cp -r dist/* /var/www/frontend/
systemctl restart nginx
```

---

# Final URLs

Frontend:

```
https://sampleProject.com
```

Backend API:

```
https://app.sampleProject.com
```

---

# Server Structure

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

# Done ✅

Your production server is now fully deployed.
