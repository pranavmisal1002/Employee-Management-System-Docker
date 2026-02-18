# 🚀 Employee Management System – Dockerized Deployment on AWS

This project demonstrates a production-style deployment of a full-stack Employee Management System using:

- 🖥 **Backend:** Spring Boot (Dockerized)
- 🌐 **Frontend:** React + Nginx (Dockerized)
- 🛢 **MySQL:** AWS RDS
- 🍃 **MongoDB:** MongoDB Atlas
- ☁ **Infrastructure:** AWS EC2

---

## 📚 Prerequisites

- AWS Account
- EC2 Instance (Ubuntu 22.04)
- RDS MySQL instance
- MongoDB Atlas cluster
- Docker installed
- Git installed

---

## 🔷 PHASE 1: Database Setup

### ✅ Step 1: Create AWS RDS (MySQL)

**Configure RDS:**

- Engine: MySQL
- Database Name: `employee_management`
- Public Access: Yes
- Allow port `3306` from EC2 Security Group

**Install MySQL client on EC2:**

```bash
sudo apt update
```
```bash
sudo apt install mysql-client -y
```

**Connect to RDS:**

```bash
mysql -h <RDS_ENDPOINT> -u admin -p
```

**Create database:**

```sql
CREATE DATABASE employee_management;
```

---

### ✅ Step 2: Create MongoDB Atlas Cluster

1. Create free cluster (M0)
2. Create database user
3. Allow network access (`0.0.0.0/0` for development)
4. Copy Java connection string

**Example format:**

```
mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/employee_management
```

---

## 🔷 PHASE 2: EC2 Setup

### ✅ Step 3: Launch EC2 Instance

**Configuration:**

- AMI: Ubuntu 22.04
- Instance Type: t3.micro

**Security Group:**

- SSH (22)
- HTTP (80)
- Custom TCP (8080)

**Connect:**

```bash
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
```

---

### ✅ Step 4: Install Docker

```bash
sudo apt update
```
```bash
sudo apt install docker.io -y
```
```bash
sudo systemctl start docker
```
```bash
sudo systemctl enable docker
```

**Verify:**

```bash
docker --version
```

---

## 🔷 PHASE 3: Backend Deployment (Dockerized Spring Boot)

### ✅ Step 5: Clone Repository

```bash
git clone https://github.com/Rohit-1920/Employee-Management-Fullstack-App.git
```
```bash
cd Employee-Management-Fullstack-App/backend
```

---

### ✅ Step 6: Configure Database (External Configuration)

The application properties values are being loaded from an external config.properties file, so please do not edit the application.properties file.
> ⚠️ **Do NOT edit `application.properties`.**

**Create external configuration file(/backend/config.properties):**

```bash
nano config.properties
```

**Paste and modify with your credentials:**

```properties
# MySQL Configuration
MYSQL_HOST=<YOUR_RDS_ENDPOINT>
MYSQL_PORT=3306
MYSQL_DB=employee_management
MYSQL_USER=admin
MYSQL_PASSWORD=<YOUR_PASSWORD>
MYSQL_SSL_MODE=REQUIRED

# MongoDB Configuration
MONGO_URI=<YOUR_MONGODB_ENDPOINT_COPY_FROM_MONGOBD_ATLAS>
```

---

### ✅ Step 7: Create Backend Dockerfile
A Dockerfile is already present in this directory, so please delete it or replace its content with the following Dockerfile content.

```bash
nano Dockerfile
```

**Paste:**

```dockerfile
# -------- Stage 1: Build --------
FROM maven:3.9.6-eclipse-temurin-11 AS build

WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# -------- Stage 2: Run --------
FROM eclipse-temurin:11-jre

WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```

---

### ✅ Step 8: Build Backend Docker Image

```bash
docker build -t employee-backend .
```

**Verify image:**

```bash
docker images
```

---

### ✅ Step 9: Run Backend Container

```bash
docker run -d -p 8080:8080 --env-file config.properties employee-backend
```

**Verify container:**

```bash
docker ps
```

---

### 🔎 Verify Backend

**Open in browser:**

```
http://<EC2_PUBLIC_IP>:8080/swagger-ui.html
```

✅ **Backend is successfully deployed.**

---

## 🔷 PHASE 4: Frontend Deployment (Dockerized React + Nginx)

### ✅ Step 10: Move to Frontend Directory

```bash
cd ../frontend
```

---

### ✅ Step 11: Configure Environment File

```bash
nano .env
```

**Update backend API URL:**

```env
REACT_APP_API_URL=http://<EC2_PUBLIC_IP>:8080/api
```

> ⚠️ **React reads environment variables at build time.**

---

### ✅ Step 12: Configure Nginx for React Routing

**Create nginx directory:**
Create a directory in the front-end named nginx. Inside the nginx directory, create a file named default.conf and paste the following content into it.

```bash
mkdir nginx
```
```bash
nano nginx/default.conf
```

**Paste:**
This configuration enables React routing.

```nginx
server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}

```

---

### ✅ Step 13: Create Frontend Dockerfile
A Dockerfile is already present in this directory, so please delete it or replace its content with the following Dockerfile content.
```bash
nano Dockerfile
```

**Paste:**

```dockerfile
# -------- Stage 1: Build --------
FROM node:18-alpine AS build

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# -------- Stage 2: Serve --------
FROM nginx:alpine

RUN rm /etc/nginx/conf.d/default.conf
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

### ✅ Step 14: Build Frontend Image

```bash
docker build -t employee-frontend .
```

> ⚠️ **You may see npm warnings about deprecated versions.**  
> This does not affect container functionality.

**Verify image:**

```bash
docker images
```

---

### ✅ Step 15: Run Frontend Container

```bash
docker run -d -p 80:80 employee-frontend
```

**Verify:**

```bash
docker ps
```

---

## 🌍 Access Application

**Frontend:**

```
http://<EC2_PUBLIC_IP>
```

**Backend:**

```
http://<EC2_PUBLIC_IP>:8080
```

---

## 🔐 Login / Registration

- ❌ **No default username or password**
- ✅ **Users must REGISTER first**
- ✅ **After registration, users can LOGIN**
- 📦 **User credentials are stored securely in the database**

---

## 🎉 Deployment Completed Successfully
