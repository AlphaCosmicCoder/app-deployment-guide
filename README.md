# 🌐 Full-Stack Deployment on Amazon EC2 (React + Node.js + MongoDB Atlas)

This beginner-friendly guide walks you through deploying a full-stack web application named **MyApp** using React (frontend), Node.js (backend), MongoDB Atlas (database), and Amazon EC2 (server). The deployment uses Nginx as a web server, PM2 to keep the backend alive, and HTTPS via Let's Encrypt. You’ll also use your custom domain for secure access.

---

## 🧱 Tech Stack
- **Frontend**: React.js
- **Backend**: Node.js (Express.js framework)
- **Database**: MongoDB Atlas (cloud-based)
- **Server**: Amazon EC2 with Ubuntu 22.04
- **Web Server**: Nginx
- **Process Manager**: PM2
- **SSL**: Let’s Encrypt via Certbot

---

## 1. 🚀 Launch and Configure EC2

1. **Log in to AWS Console** and go to **EC2 Dashboard**.
2. **Launch a new instance**:
   - Choose **Ubuntu Server 22.04 LTS** as your AMI.
   - Choose an instance type (e.g., **t2.micro** for free tier).
   - Create a **new key pair** or use an existing one. Download the `.pem` file and keep it safe.
3. **Configure security group**:
   - Allow **SSH (TCP 22)** to connect to your server remotely.
   - Allow **HTTP (TCP 80)** and **HTTPS (TCP 443)** to access your app.
4. **Launch the instance** and assign an **Elastic IP** to make the server address static.
5. **SSH into your instance**:
   ```bash
   ssh -i /path/to/your-key.pem ubuntu@your-ec2-public-ip
   ```

---

## 2. ⚙️ Install Required Software

Update system and install basic tools:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl ufw nginx
```

Install Node.js and PM2:
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g pm2
```

Configure firewall:
```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw --force enable
```

---

## 3. 🧠 Deploy Node.js Backend (MyApp Server)

Clone your backend code:
```bash
cd ~
git clone <your-backend-repo-url> myapp-backend
cd myapp-backend
npm install
```

Set up environment variables:
- Create a `.env` file or use `process.env` to include:
  - `MONGODB_URI`: Your MongoDB Atlas URI
  - `PORT`: Backend port (e.g., 3000)

Start the server:
```bash
pm2 start index.js --name myapp-backend
```

This keeps your Node.js backend running even after a reboot.

---

## 4. 💅 Deploy React Frontend (MyApp Client)

Clone and build your frontend:
```bash
cd ~
git clone <your-frontend-repo-url> myapp-frontend
cd myapp-frontend
npm install
npm run build
```

Move build files to Nginx directory:
```bash
sudo mkdir -p /var/www/myapp
sudo cp -r build/* /var/www/myapp/
sudo chown -R www-data:www-data /var/www/myapp
```

This makes your React site available to Nginx for serving.

---

## 5. 🌐 Configure Nginx Reverse Proxy

Create a config file for Nginx:
```bash
sudo nano /etc/nginx/sites-available/myapp
```

Paste this:
```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/myapp;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Nginx now serves the frontend and proxies API calls to backend.

---

## 6. 🔐 Enable HTTPS with Let’s Encrypt

Install Certbot:
```bash
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Run Certbot to issue and install certificate:
```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Verify automatic renewal:
```bash
sudo certbot renew --dry-run
```

---

## 7. 🗄️ Set Up MongoDB Atlas

1. Visit [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) and create an account.
2. Create a new cluster (shared or free tier).
3. Whitelist your EC2’s IP under **Network Access**.
4. Create a database user and password.
5. Copy the connection URI and use it in your backend `.env` file:
```env
MONGODB_URI=mongodb+srv://<username>:<password>@cluster0.mongodb.net/mydatabase
```
6. Restart your backend:
```bash
pm2 restart myapp-backend
```

---

## 8. 🔁 Auto-Start Services on Reboot

To keep backend running after reboot:
```bash
pm2 startup systemd
# It will print a command, copy and run it
sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u ubuntu --hp /home/ubuntu

pm2 save
```

Nginx runs automatically on system start.

---

## ✅ Final Result

Your app **MyApp** is now live and secure at:
```
https://yourdomain.com
```
- React frontend served by Nginx
- Node.js backend via Nginx reverse proxy
- SSL certificate via Let's Encrypt
- Data stored in MongoDB Atlas

---

## 💡 Tips
- Keep your environment variables safe using `.env` files.
- Regularly pull code updates from GitHub and restart PM2.
- Monitor logs with:
```bash
pm2 logs
```

You're now equipped to deploy production-ready full-stack apps on AWS 🚀
