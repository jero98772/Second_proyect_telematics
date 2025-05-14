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


## Objectives Covered

* Objective 1: Deploy monolithic app on EC2 with NGINX reverse proxy
* Objective 2: Implement cloud scalability using Auto Scaling Group, Load Balancer, RDS, and shared storage (EFS)

---

## Technologies Used

| Technology                  | Purpose                                           |
| --------------------------- | ------------------------------------------------- |
| AWS EC2                     | Host app in the cloud                             |
| Docker                      | Run app and DB in containers                      |
| Docker Compose              | Define and orchestrate multi-container deployment |
| NGINX                       | Reverse proxy to expose app on port 80            |
| AWS AMI                     | Create reusable image of EC2 configuration        |
| Launch Template             | Define settings to auto-launch EC2 instances      |
| Auto Scaling                | Ensure redundancy and scalability                 |
| Elastic Load Balancer (ELB) | Distribute traffic across EC2 instances           |
| Amazon RDS (MySQL)          | Host managed relational DB instance               |
| Amazon EFS                  | Provide shared storage accessible by all EC2s     |

---

## Requirements

* AWS Account or Learner Lab access
* Basic Linux CLI and Docker familiarity
* AWS Console access for EC2, RDS, EFS, and Auto Scaling

---



