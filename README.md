# 3-Tier Book Management App on AWS (React + Node.js + RDS + ALB + Bastion)

A production-style 3-Tier AWS deployment demonstrating secure communication between frontend, backend, and database tiers using Amazon EC2, Application Load Balancers, RDS, Security Groups, and a Bastion Host.

---

## 🏗️ Architecture Overview

- **Public Subnets**: Bastion host & Frontend ALB  
- **Private Subnets**: Frontend, Backend, RDS  
- **Security Groups**:
  | SG | Allows Inbound From | Purpose |
  |----|----------------------|----------|
  | Bastion-SG | SSH (My IP) | Jump access |
  | ALB-Frontend-SG | 0.0.0.0/0 | Internet traffic |
  | Frontend-SG | ALB-Frontend-SG, Bastion-SG | Web + SSH |
  | Backend-SG | Frontend-SG | API access |
  | RDS-SG | Backend-SG | DB connection |

---

## ⚙️ Tech Stack

| Layer | Technology | Description |
|--------|-------------|-------------|
| Frontend | React.js + Apache HTTPD | UI + Reverse Proxy |
| Backend | Node.js + Express + PM2 | REST APIs |
| Database | AWS RDS MySQL | Persistent storage |
| Infrastructure | AWS VPC, EC2, ALB, S3, CloudWatch | Cloud platform |

---

## 🧩 Step-by-Step Deployment

### 1️⃣ Create VPC and Subnets
- 2 Public + 6 Private subnets across 2 AZs  
- Enable Internet Gateway and NAT Gateway for outbound access.

### 2️⃣ Launch EC2 Instances
| Role | Subnet | SG | Key Tasks |
|------|---------|----|-----------|
| Bastion | Public | Bastion-SG | SSH Jumpbox |
| Frontend | Private | Frontend-SG | Host React + Apache |
| Backend | Private | Backend-SG | Run Node.js API |

### 3️⃣ Create RDS (MySQL)
- Engine: MySQL  
- Deployment: Multi-AZ  
- Accessibility: Private  
- SG: RDS-SG (allow 3306 from Backend-SG)

### 4️⃣ Create ALBs
| Name | Scheme | TG Port | Purpose |
|------|---------|---------|----------|
| Frontend-ALB | Internet-Facing | 80 | Serves React App |
| Backend-ALB | Internal | 3000 | For private backend APIs |

---

## 🖥️ Frontend Setup (React)

```bash
# SSH into frontend EC2
sudo yum install -y git nodejs httpd
git clone <your-repo-url>
cd 2nd10WeeksofCloudOps-main/client

# Update config.js
vi src/pages/config.js
export const API_BASE_URL = "http://alb-frontend-<dns>.elb.amazonaws.com/api";

npm install
npm run build
sudo cp -r build/* /var/www/html/
sudo systemctl start httpd
sudo systemctl enable httpd

```

### Apache Reverse Proxy

```bash
sudo vi /etc/httpd/conf/httpd.conf
```

### Add at the end:
```bash
<VirtualHost *:80>
    DocumentRoot /var/www/html
    ProxyRequests Off
    ProxyPreserveHost On
    ProxyPass /api/ http://internal-alb-backend-<dns>.elb.amazonaws.com/
    ProxyPassReverse /api/ http://internal-alb-backend-<dns>.elb.amazonaws.com/
</VirtualHost>
```

### Restart Apache:
```bash
sudo systemctl restart httpd
```
### ⚙️ Backend Setup (Node.js)
```bash
# SSH into backend EC2
sudo yum install -y git nodejs
git clone <your-repo-url>
cd backend

# Configure Environment
vi .env
DB_HOST=database-1.<your-rds-endpoint>.us-east-1.rds.amazonaws.com
DB_USERNAME=admin
DB_PASSWORD="YourRdsPassword"
PORT=3000

npm install
npm install pm2 -g

# Start Backend
pm2 start index.js --name "backendApi"
pm2 save
```
## Test from backend EC2:

```bash
curl http://localhost:3000/books
```

### 🗄️ Database
SSH into backend EC2 → connect to RDS:
```bash
mysql -h <rds-endpoint> -u admin -p
CREATE DATABASE books;
USE books;
CREATE TABLE books (id INT AUTO_INCREMENT PRIMARY KEY, title VARCHAR(100), `desc` TEXT, price FLOAT, cover VARCHAR(255));

```
## 🚀 Final Test
Open in browser:
```bash
http://alb-frontend-<dns>.elb.amazonaws.com
```

🧑‍💻 Author

Anirudh Trivedi

Multi-Cloud | DevOps | AWS 
Connect on LinkedIn 🚀 -> https://www.linkedin.com/in/anirudh-trivedi-4414b9244/
 



