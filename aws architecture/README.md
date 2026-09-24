# Construction Technect – AWS Deployment Architecture

## Overview
Construction Technect is a SaaS application built with:
- **Frontend:** Next.js
- **Backend:** Nest.js
- **Database:** PostgreSQL

This document describes a basic AWS deployment architecture for the application.

## Architecture Diagram


![AWS Architecture Diagram](architecture-diagram.png)



## AWS Services Selected

| Service | Purpose | Why This Service |
|---|---|---|
| Route 53 | DNS | Routes the domain name to the application, integrates natively with other AWS services |
| CloudFront | CDN | Delivers the frontend fast globally with caching, and provides free HTTPS |
| Amplify Hosting | Frontend hosting | Native support for Next.js (SSR + static), simple Git-based CI/CD |
| Application Load Balancer (ALB) | Traffic routing | Distributes incoming API requests across backend containers, performs health checks |
| ECS Fargate | Backend compute | Runs the Nest.js backend in containers without managing servers |
| ECR | Container registry | Stores the Docker image for the Nest.js backend |
| RDS (PostgreSQL) | Database | Fully managed PostgreSQL with automated backups and Multi-AZ failover |
| Secrets Manager | Credential storage | Stores database credentials and API keys securely, outside the codebase |
| CloudWatch | Monitoring | Collects logs, metrics, and triggers alarms for ECS and RDS |
| ACM (Certificate Manager) | SSL/TLS | Provides free, auto-renewing HTTPS certificates for CloudFront and ALB |
| VPC (Private Subnet) | Network isolation | Keeps the database completely inaccessible from the public internet |

## Architecture Explanation

### 1. Frontend Deployment
The Next.js frontend is deployed on **AWS Amplify Hosting**, connected directly to the Git repository. Amplify automatically builds and deploys on every push, and supports Next.js SSR out of the box.

### 2. Backend Deployment
The Nest.js backend is containerized with Docker, pushed to **ECR**, and run on **ECS Fargate** — a serverless container service that removes the need to manage EC2 instances.

### 3. Database Hosting
PostgreSQL is hosted on **Amazon RDS**, configured as Multi-AZ for high availability, and placed inside a **private subnet** with no public IP address.

### 4. User Access
Users access the app through a domain managed by **Route 53**, which routes traffic to **CloudFront** (serving the frontend) and the **Application Load Balancer** (routing API calls to the backend). Both endpoints use HTTPS via **ACM** certificates.

### 5. Backend–Database Communication
ECS Fargate tasks and the RDS instance sit inside the same **VPC**. The backend connects to the database over the private internal network on port 5432 — this traffic never touches the public internet.

### 6. Database Security
- No public IP on RDS — unreachable from outside the VPC
- Security Group only allows inbound traffic from the ECS backend's security group
- Data encrypted at rest (KMS) and in transit (SSL)
- Credentials stored in Secrets Manager, never hardcoded

### 7. Monitoring
**CloudWatch** collects logs and metrics (CPU, memory, request counts) from ECS Fargate and RDS. **CloudWatch Alarms** can trigger notifications via **SNS** when thresholds are breached (e.g. high CPU or low storage).

### 8. Database Backups
RDS performs **automated daily backups** with point-in-time recovery, plus manual **snapshots** before major deployments or schema changes.

### 9. Secrets and Credentials Management
All sensitive values (DB username/password, API keys) are stored in **AWS Secrets Manager**. ECS task definitions reference these secrets by ARN and inject them as environment variables at runtime — they are never stored in code or Docker images.

### 10. Scaling Strategy
- **ECS Fargate** auto-scales the number of running containers based on CPU/memory/request load
- **ALB** automatically load-balances traffic across all running containers
- **RDS** can add read replicas if read traffic increases
- **CloudFront** caching reduces load on the origin servers
- **Amplify** scales the frontend automatically with no extra configuration

## Note
This is a design-only submission. The complete architecture has not been deployed, as permitted by the assignment instructions.
