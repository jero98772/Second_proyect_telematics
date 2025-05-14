# Members:
- Felipe Uribe Correa
- Laura Danniela Zárate Guerrero
- Victor Daniela Arango Sohm

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

## ⚙️ Configuration Summary and Executed Steps

This section summarizes all the configurations applied and steps executed to deploy and scale the BookStore application on AWS infrastructure, fulfilling Objectives 1 and 2 of the project.

### 1. EC2 Base Instance

* OS: Ubuntu 22.04 LTS
* Instance type: t2.micro
* Installed: Docker, Docker Compose, NGINX, mysql-client, nfs-common
* Security Group Inbound Rules:

  * TCP 22 (SSH)
  * TCP 80 (HTTP)
  * TCP 5000 (Flask development)
  * TCP 3306 (MySQL RDS)
  * TCP 2049 (EFS)

### 2. Application Setup

* BookStore app deployed with Docker Compose:

  * flaskapp (Flask + SQLAlchemy)
  * db (MySQL 8, used only locally)
* NGINX reverse proxy routes port 80 to Flask’s port 5000
* Flask SQLAlchemy was configured to connect to RDS in production

### 3. Create AMI

* From EC2 -> Instance -> Actions -> Create Image
* Issue: initial IAM role denied AMI creation
* Fix: permissions updated to allow creation.

### 4. Create Launch Template

* AMI: custom image created above
* Instance type: t2.micro
* Key pair: same as original
* Network and Security Group: same as base EC2

### 5. Create Auto Scaling Group

* Minimum: 2 instances / Maximum: 4
* Subnets: us-east-1a, 1e, 1f
* Launch Template: selected
* Load Balancer: created Application Load Balancer (ALB)
* Target Group: created and attached
* Listener: HTTP on port 80 -> forwards to target group

### 6. Configure Amazon RDS 

* Engine: MySQL 8.0
* Instance type: db.t3.micro
* Public access: enabled 
* Username: admin / Password: \ secure_password
* Connected from EC2:

```bash
mysql -h <rds-endpoint> -u admin -p
CREATE DATABASE bookstore;
EXIT;
```

* Flask app.py connection string:

```python
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://admin:<pwd>@<endpoint>/bookstore'
```

* Restarted app with:

```bash
sudo docker-compose down
sudo docker-compose up --build -d
```

### 7. Create and Mount Amazon EFS

* EFS created in the same VPC
* Mount targets in subnets us-east-1a, 1e, and 1f
* Security Group for EFS allowed TCP 2049 from EC2 SG or VPC CIDR
* On EC2:

```bash
sudo apt install -y nfs-common
sudo mkdir /mnt/bookstore-efs
sudo mount -t nfs4 -o nfsvers=4.1 <efs-dns>:/ /mnt/bookstore-efs
```

* Test:

```bash
df -h | grep efs
echo "EFS OK!" | sudo tee /mnt/bookstore-efs/test.txt
cat /mnt/bookstore-efs/test.txt
```

### 8. Custom Domain Integration (salchicha.space)

* Domain registered via Namecheap
* Attempted to use Route 53, but IAM policy denied:

  * route53\:CreateHostedZone
  * acm\:RequestCertificate
* Used Cloudflare for DNS and SSL proxy:

  * Configured Cloudflare nameservers in Namecheap
  * CNAME record: [www.salchicha.space](http://www.salchicha.space) -> bookstore-alb-xxxxxx.us-east-1.elb.amazonaws.com
  * Enabled Full SSL mode and “Always Use HTTPS”

Due to subsequent IAM restrictions in the lab environment, access to EC2, Target Groups, and Load Balancer was lost. This prevented final Cloudflare validation (error 521: connection refused).

---

## What we accomplished:

* The application is now deployed across multiple EC2 instances
* Auto scaling enabled to meet load demands
* Traffic distributed via Application Load Balancer
* Centralized persistent database hosted on RDS
* Shared storage between instances enabled via EFS
* NGINX handles reverse proxying on each instance

---

