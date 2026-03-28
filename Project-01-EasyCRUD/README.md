# Project 01 — EASYCRUD Application

A full-stack CRUD web application built with MariaDB, Spring Boot, and React.

---

## 🏗️ Architecture

[Browser] → React Frontend (Port 80)
                  ↓
         Spring Boot API (Port 8080)
                  ↓
         MariaDB Database (Port 3306)

---

## 📁 Project Structure

project-01-easycrud/
├── database/     → MariaDB setup guide
├── backend/      → Spring Boot REST API
├── frontend/     → React (Vite) frontend
└── README.md

---

## 📦 Tech Stack

| Layer    | Technology        |
|----------|-------------------|
| Frontend | React, Vite       |
| Backend  | Spring Boot, Java |
| Database | MariaDB           |
| Server   | Apache2, Ubuntu   |

---

## 🗄️ Part 1: Database Deployment (MariaDB)

### Step 1 — Install MariaDB
```bash
apt update && apt install mariadb-server -y
```

### Step 2 — Secure the Installation
```bash
mysql_secure_installation
```
Follow prompts to set root password, remove test databases, disable remote root login.

### Step 3 — Login to MariaDB
```bash
mysql -h database-1.cto44y8iy755.ap-south-1.rds.amazonaws.com -u Saroj -p
```

### Step 4 — Create Database and User
```sql
CREATE DATABASE student_db;

EXIT;
```

---

## ☕ Part 2: Backend Deployment (Spring Boot)

### Step 1 — Install Java 17
```bash
apt update && apt install openjdk-17-jdk -y
java -version
```

### Step 2 — Install Maven
```bash
apt install maven -y
mvn -version
```

### Step 3 — Configure Database Connection
```bash
vim backend/src/main/resources/application.properties
```
```properties
server.port=8080
spring.datasource.url=jdbc:mariadb://<DB_HOST>:3306/student_db
spring.datasource.username=<DB_USER>
spring.datasource.password=<DB_PASS>
```

### Step 4 — Build the Application
```bash
mvn clean package
```

### Step 5 — Run the Application
```bash
java -jar target/student-registration-backend-0.0.1-SNAPSHOT.jar
```

### Step 6 — Run in Background
```bash
nohup java -jar target/student-registration-backend-0.0.1-SNAPSHOT.jar > app.log 2>&1 &
```

> Application accessible at: `http://localhost:8080`

---

## ⚛️ Part 3: Frontend Deployment (React + Vite)

### Step 1 — Install Node.js and npm
```bash
apt update && apt install nodejs npm -y
node -v
npm -v
```

### Step 2 — Install Project Dependencies
```bash
npm install
```

### Step 3 — Configure Backend URL
```bash
vim .env
```
```env
VITE_API_URL="http://<BACKEND_PUBLIC_IP>:8080/api"
```

### Step 4 — Build for Production
```bash
npm run build
```

### Step 5 — Deploy with nginx
```bash
apt install nginx -y
systemctl start nginx
cp -rf dist/* /var/www/html/
```

> Application accessible at: `http://localhost:8080`

---

## 👤 Author
**Saroj Behera** | Project 01 Submission