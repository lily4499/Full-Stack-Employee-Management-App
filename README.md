
# 🚀 Employee Management App – Production Deployment Guide

## 🌍 Real-World Scenario

You’re a DevOps Engineer deploying a full-stack **Employee Management System** consisting of:

- 🖥️ A Spring Boot backend (Java)
- 💻 A React frontend (Node.js)
- 🗄️ A MySQL database (AWS RDS)
- 🌐 A production web server (NGINX on Ubuntu)
- 🔐 SSL with Let's Encrypt (Certbot)

This guide walks you through deploying the **entire stack to production**, step-by-step.

---

## 📁 Project Structure

```bash
employee-app/
├── employeemanagmentbackend/         # Spring Boot backend (Java)
│   └── target/                       # JAR files after build
├── employeemanagement-frontend/     # React frontend (Node.js)
│   └── build/                        # Production build folder
```

---

## 🧰 Step-by-Step Deployment Instructions

---

### 🖥️ Step 1: Create Production Server

**Purpose:** Set up a clean Ubuntu server for hosting backend and frontend.

```bash
# Create Ubuntu 22.04 VM (AWS EC2, DigitalOcean, etc.)

# Add a new non-root user
adduser spring
usermod -aG sudo spring
su - spring
```

---

### 🧩 Step 2: Install Required Software

**Purpose:** Prepare the server with Java, Maven, and Node.js.

```bash
sudo apt update && sudo apt upgrade -y

# Java 17
sudo apt install openjdk-17-jdk openjdk-17-jre -y
java --version

# Node.js (LTS)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Maven
sudo apt install maven -y
```

---

### 🐳 Step 3: Clone and Build Backend App

**Purpose:** Build and deploy the Spring Boot backend.

```bash
git clone https://github.com/lily4499/employee-app.git
cd employee-app/employeemanagmentbackend
mvn clean install
```

---

### ⚙️ Step 4: Create a systemd Service for Backend

**Purpose:** Manage Spring Boot app as a background service.

```bash
sudo nano /etc/systemd/system/spring.service
```

Paste this (update paths as needed):

```ini
[Unit]
Description=Spring Boot Backend Service
After=syslog.target

[Service]
User=spring
Restart=always
RestartSec=30s
ExecStart=/usr/bin/java -jar /home/spring/employee-app/employeemanagmentbackend/target/employeemanagmentbackend-0.0.1-SNAPSHOT.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable spring.service
sudo systemctl start spring.service
```

---

### 🗄️ Step 5: Set Up MySQL Database (RDS)

**Purpose:** Store persistent employee data.

1. Go to **AWS Console → RDS → Create MySQL Database**.
2. Take note of:
   - Endpoint
   - Port
   - DB name
   - Username/password

**➡️ Edit `application.properties`:**

```properties
spring.datasource.url=jdbc:mysql://<RDS_ENDPOINT>:3306/<DB_NAME>
spring.datasource.username=<your_db_user>
spring.datasource.password=<your_db_password>
```

Rebuild and restart the backend:

```bash
mvn clean install
sudo systemctl restart spring.service
```

---

### 🌐 Step 6: Build and Deploy Frontend (React)

**Purpose:** Generate static frontend files and serve them via NGINX.

```bash
cd ~/employee-app/employeemanagement-frontend
npm install
npm run build

sudo mkdir -p /var/www/front
sudo cp -r build/* /var/www/front
```

---

### 🌍 Step 7: Configure NGINX

**Purpose:** Serve frontend and reverse proxy to backend.

```bash
sudo nano /etc/nginx/sites-available/spring
```

Paste below:

```nginx
server {
 listen 80;
 server_name lilianedevops.online www.lilianedevops.online;

 location / {
   root /var/www/front;
   index index.html;
   try_files $uri $uri/ /index.html;
 }
}

server {
 listen 80;
 server_name spring.lilianedevops.online;

 location / {
   proxy_pass http://127.0.0.1:8080;
   proxy_http_version 1.1;
   proxy_set_header Upgrade $http_upgrade;
   proxy_set_header Connection 'upgrade';
   proxy_set_header Host $host;
   proxy_cache_bypass $http_upgrade;
 }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/spring /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

---

### 🔐 Step 8: Secure with Certbot SSL

**Purpose:** Enable HTTPS via Let's Encrypt.

```bash
sudo snap install core; sudo snap refresh core
sudo apt remove certbot -y
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

sudo certbot --nginx -d lilianedevops.online -d spring.lilianedevops.online
```

---

### 🛠️ Step 9: Update API URL in Frontend

**Purpose:** Point React frontend to HTTPS backend endpoint.

```bash
cd ~/employee-app/employeemanagement-frontend/src/service
nano EmployeeService.js
```

Update:

```js
const EMPLOYEE_API_BASE_URL = "https://spring.lilianedevops.online/api/employees";
```

---

## ✅ Final Verification

- ✅ Visit `https://lilianedevops.online` → React App loads
- ✅ Visit `https://spring.lilianedevops.online` → Backend API accessible
- ✅ Confirm frontend connects to backend via browser Dev Tools

🧪 Steps to confirm Frontend ↔ Backend Connection:
Open the frontend app in your browser:  
https://lilianedevops.online
Open Developer Tools:  
Press F12 or Ctrl + Shift + I (or right-click → "Inspect").  
Go to the Network tab.  
Refresh the page (F5) and watch for API calls being made to the backend:
 - Look for a request like:
```GET https://spring.lilianedevops.online/api/employees
```
 - Status should be 200 OK.

Click on the request to view details:
 - Check the Response tab to verify that data (JSON list of employees) is returned.
 - Check the Headers tab to confirm it’s connecting to the backend domain.



---

## 🧾 Summary

| Component  | Purpose                          | Technology          |
|------------|----------------------------------|---------------------|
| Backend    | Employee API                     | Spring Boot (Java)  |
| Frontend   | User Interface                   | React (Node.js)     |
| Database   | Persistent data store            | AWS RDS (MySQL)     |
| Web Server | Serve frontend & proxy backend   | NGINX               |
| Security   | HTTPS encryption & auto-renewal  | Certbot (Let's Encrypt) |
| Systemd    | Manage Spring app as a service   | systemd             |

---

