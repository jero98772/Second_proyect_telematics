# BookStore AWS Deployment

This project demonstrates a scalable deployment of a monolithic Python Flask application using AWS infrastructure, including:

- EC2 + Docker + NGINX
- Auto Scaling Group
- Load Balancer (ALB)
- Amazon RDS for MySQL
- Amazon EFS
- Custom Domain via Cloudflare

## Structure
- `app.py` – Flask application
- `Dockerfile` – App containerization
- `docker-compose.yml` – Service definition
- `infra/` – Infrastructure setup notes

---

## Project Description

BookStore is a monolithic e-commerce system that enables users to sell and purchase second-hand books. The application includes:

* User authentication (register, login, logout)
* Catalog visualization
* Book purchase (stock management)
* Simulated payment
* Simulated delivery

Initially deployed as two Docker containers (Python Flask + MySQL) using Docker Compose on a single VM.

---

## Objectives Covered

* Objective 1: Deploy monolithic BookStore on AWS EC2 with a domain, SSL certificate, and NGINX reverse proxy.
* Objective 2: Scale the monolithic app in AWS using Auto Scaling, external RDS database, and shared file system (EFS or NFS).

---

## Technologies Used

| Component      | Technology                                | Purpose                             |
| -------------- | ----------------------------------------- | ----------------------------------- |
| Cloud Provider | AWS                                       | Hosting infrastructure              |
| Compute        | EC2 (Ubuntu 22.04, t2.micro)              | Base instance for deployment        |
| Containers     | Docker, Docker Compose                    | Containerized app + DB              |
| Proxy          | NGINX                                     | Expose app via HTTP (port 80)       |
| Image          | AMI                                       | Custom image for auto scaling       |
| Scaling        | Launch Template, Auto Scaling Group (ASG) | Define and scale instances          |
| Load Balancer  | Application Load Balancer (ALB/ELB)       | Traffic distribution and failover   |
| Database       | Amazon RDS (MySQL 8.0)                    | External managed database           |
| Shared Storage | Amazon EFS                                | Shared file system across instances |

---

## Configuration Summary

### 1. EC2 Base Instance:

* Ubuntu 22.04 LTS
* Installed Docker, Docker Compose, and NGINX
* Security Group Inbound Rules:

  * TCP 22 (SSH) – from trusted IP
  * TCP 80 (HTTP) – from 0.0.0.0/0
  * TCP 5000 (Flask direct, optional)
  * TCP 3306 (RDS, restricted by CIDR or SG)
  * TCP 2049 (EFS, restricted by CIDR or SG)

### 2. Docker Application:

* docker-compose.yml with services:

  * flaskapp: Python + Flask + SQLAlchemy
  * db: MySQL 8 (used only during local phase)
* Flask configured to connect to RDS in production

### 3. NGINX Reverse Proxy:

* Forward requests from port 80 to container on port 5000
* Configured under /etc/nginx/sites-available/bookstore

### 4. RDS:

* Engine: MySQL 8.0
* Instance type: db.t3.micro
* Public access: Enabled
* Database: bookstore
* Connection from EC2 via private VPC or public IP (for testing)
* SQLAlchemy connection string in Flask:

  'mysql+pymysql://admin\:your\_password\@bookstore-db.rds.amazonaws.com/bookstore'

### 5. EFS:

* Created in same VPC and subnets as EC2s
* Mount targets in 3 AZs
* Mounted in all instances at /mnt/bookstore-efs
* Tested with echo and cat

### 6. AMI:

* Image created from fully configured EC2
* Includes app, Docker, NGINX, mounts, etc.

### 7. Launch Template:

* Based on AMI
* Type: t2.micro
* Key pair and SG same as original instance

### 8. Auto Scaling Group:

* Min: 2, Max: 4
* Subnets: 3 across us-east-1a, 1e, 1f
* Health checks via ELB
* Attached to ALB (Application Load Balancer)

### 9. Application Load Balancer:

* Listener on port 80
* Forward to target group attached to ASG instances
* DNS used to access live service

---

## What we accomplished:

* The application is now deployed across multiple EC2 instances
* Auto scaling enabled to meet load demands
* Traffic distributed via Application Load Balancer
* Centralized persistent database hosted on RDS
* Shared storage between instances enabled via EFS
* NGINX handles reverse proxying on each instance

---

