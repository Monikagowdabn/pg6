# AWS Containerized Application Deployment & Operations Guide
> **ECR • ECS • EKS • RDS Database Integration**

---

## Table of Contents
- [Task 1 — Application & ECR](#task-1--application--ecr)
- [Task 2 — Deploy Using Amazon ECS](#task-2--deploy-using-amazon-ecs)
- [Task 3 — Deploy Using Amazon EKS](#task-3--deploy-using-amazon-eks)
- [Task 4 — Database Integration](#task-4--database-integration)
- [Security Best Practices](#security-best-practices)

---

## Task 1 — Application & ECR

### 1.1 Environment Setup

Install and start Docker on the EC2 instance:

```bash
# Update system packages
sudo yum update -y

# Install Docker
sudo yum install docker -y

# Start Docker service
sudo service docker start

# (Optional) Add current user to docker group
sudo usermod -aG docker ec2-user
```

---

### 1.2 Create the Dockerfile

```bash
touch Dockerfile
nano Dockerfile
```

**Dockerfile contents:**

```dockerfile
FROM public.ecr.aws/amazonlinux/amazonlinux:latest

RUN yum update -y && \
    yum install -y httpd

RUN echo 'Hello World!' > /var/www/html/index.html

RUN echo 'mkdir -p /var/run/httpd' >> /root/run_apache.sh && \
    echo 'mkdir -p /var/lock/httpd' >> /root/run_apache.sh && \
    echo '/usr/sbin/httpd -D FOREGROUND' >> /root/run_apache.sh && \
    chmod 755 /root/run_apache.sh

EXPOSE 80

CMD /root/run_apache.sh
```

---

### 1.3 Build & Test Docker Image

```bash
# Build the image
docker build -t hello-world .

# Verify the image was created
docker images --filter reference=hello-world

# Test locally on port 80
docker run -t -i -p 80:80 hello-world
```

> ✅ **Result:** Navigating to `http://localhost` displays **Hello World!**

---

### 1.4 Push Image to Amazon ECR

```bash
# Step 1: Create the ECR repository
aws ecr create-repository \
  --repository-name hello-repository \
  --region us-east-1
```

**ECR Repository created successfully:**

| Field | Value |
|---|---|
| Repository ARN | `arn:aws:ecr:us-east-1:863942760608:repository/hello-repository` |
| Registry ID | `863942760608` |
| Repository URI | `863942760608.dkr.ecr.us-east-1.amazonaws.com/hello-repository` |
| Region | `us-east-1` |
| Encryption | AES-256 |
| Created | 2026-03-17T11:48:42Z |
| Tag Mutability | MUTABLE |

```bash
# Step 2: Tag the local image with the ECR URI
docker tag hello-world \
  863942760608.dkr.ecr.us-east-1.amazonaws.com/hello-repository

# Step 3: Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  863942760608.dkr.ecr.us-east-1.amazonaws.com

# Step 4: Push the image
docker push 863942760608.dkr.ecr.us-east-1.amazonaws.com/hello-repository
```

---

## Task 2 — Deploy Using Amazon ECS

### 2.1 ECS Architecture Overview

| Component | Configuration |
|---|---|
| Cluster | ECS cluster in default VPC across multiple AZs |
| Task Definition | References ECR image, exposes port 80 |
| Service | Desired count ≥ 1, rolling update strategy |
| Load Balancer | Application Load Balancer (ALB) on port 80 |
| Target Group | Health check on `/` returning HTTP 200 |
| Security Group | Inbound: TCP 80 from `0.0.0.0/0` |

---

### 2.2 Create ECS Task Definition

```bash
aws ecs register-task-definition \
  --family hello-task \
  --network-mode awsvpc \
  --requires-compatibilities FARGATE \
  --cpu 256 --memory 512 \
  --container-definitions '[{
    "name": "hello-container",
    "image": "863942760608.dkr.ecr.us-east-1.amazonaws.com/hello-repository:latest",
    "portMappings": [{"containerPort": 80, "protocol": "tcp"}],
    "essential": true
  }]'
```

---

### 2.3 Create ECS Cluster & Service

```bash
# Create ECS Cluster
aws ecs create-cluster --cluster-name hello-cluster

# Create ECS Service with ALB
aws ecs create-service \
  --cluster hello-cluster \
  --service-name hello-service \
  --task-definition hello-task \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration 'awsvpcConfiguration={
    subnets=[subnet-xxxxx,subnet-yyyyy],
    securityGroups=[sg-xxxxx],
    assignPublicIp=ENABLED}' \
  --load-balancers 'targetGroupArn=arn:aws:...,containerName=hello-container,containerPort=80'
```

---

### 2.4 Verification

> ✅ **EC2 Direct Access:** `http://34.227.65.182` → **Hello World!**
>
> ✅ **Load Balancer URL:** `http://balanceecs-2062376782.us-east-1.elb.amazonaws.com` → **Hello World!**

Both endpoints confirmed serving the containerized application successfully.

---

## Task 3 — Deploy Using Amazon EKS

### 3.1 ECS vs EKS Comparison

| Feature | ECS | EKS |
|---|---|---|
| Orchestration | AWS proprietary | Kubernetes (open standard) |
| Portability | AWS-only | Portable to any K8s cluster |
| Learning Curve | Lower | Higher |
| Control Plane Cost | Free | $0.10/hr per cluster |
| Best For | Simple AWS workloads | Complex, multi-cloud apps |

---

### 3.2 Install Prerequisites

```bash
# Install kubectl
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.28.0/2023-09-14/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --client

# Install eksctl
curl --silent --location \
  "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" \
  | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

---

### 3.3 Create EKS Cluster

```bash
# Create EKS cluster (takes ~15-20 minutes)
eksctl create cluster \
  --name hello-eks-cluster \
  --region us-east-1 \
  --nodegroup-name hello-nodes \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed

# Verify cluster nodes are ready
kubectl get nodes
```

---

### 3.4 Deploy Application to EKS

Create the Kubernetes manifest file:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-container
        image: 863942760608.dkr.ecr.us-east-1.amazonaws.com/hello-repository:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hello-service
spec:
  type: LoadBalancer
  selector:
    app: hello-world
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

Apply and verify:

```bash
# Apply the manifests
kubectl apply -f deployment.yaml

# Monitor rollout
kubectl rollout status deployment/hello-deployment

# Get the external LoadBalancer URL
kubectl get service hello-service

# Verify pods are running
kubectl get pods -o wide
```

---

### 3.5 Configure ECR Access for EKS Nodes

```bash
aws iam attach-role-policy \
  --role-name eksctl-hello-eks-cluster-nodegroup-NodeInstanceRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
```

---

### 3.6 Cleanup EKS Resources

```bash
# Delete Kubernetes resources
kubectl delete -f deployment.yaml

# Delete the EKS cluster (avoid ongoing charges)
eksctl delete cluster --name hello-eks-cluster --region us-east-1
```

---

## Task 4 — Database Integration

### 4.1 Security Architecture

| Layer | Configuration |
|---|---|
| RDS Placement | Private subnet — NOT publicly accessible |
| Access Control | Only from ECS/EKS task security group |
| Encryption at Rest | AES-256 |
| Encryption in Transit | SSL/TLS enforced |
| Credentials | Stored in AWS Secrets Manager |
| Backups | Automated with 7-day retention |

---

### 4.2 Create RDS Database

```bash
# Create DB subnet group (using private subnets)
aws rds create-db-subnet-group \
  --db-subnet-group-name hello-db-subnet \
  --db-subnet-group-description "Subnet group for Hello app RDS" \
  --subnet-ids subnet-private-1 subnet-private-2

# Create RDS MySQL instance
aws rds create-db-instance \
  --db-instance-identifier hello-db \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --engine-version 8.0 \
  --master-username admin \
  --master-user-password MySecurePass123! \
  --allocated-storage 20 \
  --db-subnet-group-name hello-db-subnet \
  --vpc-security-group-ids sg-rds-xxxxx \
  --backup-retention-period 7 \
  --storage-encrypted \
  --no-publicly-accessible
```

---

### 4.3 Store Credentials in Secrets Manager

```bash
aws secretsmanager create-secret \
  --name hello-db-credentials \
  --description "RDS credentials for Hello app" \
  --secret-string '{
    "username": "admin",
    "password": "MySecurePass123!",
    "host": "hello-db.xxxx.us-east-1.rds.amazonaws.com",
    "port": "3306",
    "dbname": "helloapp"
  }'
```

---

### 4.4 Update Dockerfile for Database Support

```dockerfile
FROM public.ecr.aws/amazonlinux/amazonlinux:latest

RUN yum update -y && \
    yum install -y httpd php php-mysqlnd

COPY index.php /var/www/html/index.php

RUN echo 'mkdir -p /var/run/httpd' >> /root/run_apache.sh && \
    echo 'mkdir -p /var/lock/httpd' >> /root/run_apache.sh && \
    echo '/usr/sbin/httpd -D FOREGROUND' >> /root/run_apache.sh && \
    chmod 755 /root/run_apache.sh

EXPOSE 80
CMD /root/run_apache.sh
```

**Application code (`index.php`) — reads credentials from environment:**

```php
<?php
$host     = getenv('DB_HOST');
$username = getenv('DB_USER');
$password = getenv('DB_PASS');
$dbname   = getenv('DB_NAME');

$conn = new mysqli($host, $username, $password, $dbname);
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Hello World! Database connected successfully.";
$conn->close();
?>
```

---

### 4.5 Update ECS Task Definition with Secrets Injection

```bash
aws ecs register-task-definition \
  --family hello-task-db \
  --network-mode awsvpc \
  --requires-compatibilities FARGATE \
  --cpu 256 --memory 512 \
  --execution-role-arn arn:aws:iam::863942760608:role/ecsTaskExecutionRole \
  --container-definitions '[{
    "name": "hello-container",
    "image": "863942760608.dkr.ecr.us-east-1.amazonaws.com/hello-repository:latest",
    "portMappings": [{"containerPort": 80}],
    "secrets": [
      {"name": "DB_HOST", "valueFrom": "arn:aws:secretsmanager:...:hello-db-credentials:host::"},
      {"name": "DB_USER", "valueFrom": "arn:aws:secretsmanager:...:hello-db-credentials:username::"},
      {"name": "DB_PASS", "valueFrom": "arn:aws:secretsmanager:...:hello-db-credentials:password::"},
      {"name": "DB_NAME", "valueFrom": "arn:aws:secretsmanager:...:hello-db-credentials:dbname::"}
    ]
  }]'
```

---

## Security Best Practices

### IAM & Access Control
- `ecsTaskExecutionRole` — allows ECS to pull from ECR and write CloudWatch logs
- `ecsTaskRole` — grants containers access to Secrets Manager, S3, etc.
- Never embed AWS credentials in Dockerfiles or application code
- Use IAM instance profiles on EC2 instead of static access keys

### Network Security
- Place RDS in **private subnets** — no direct internet access
- Restrict database security group to ECS task security group only
- Enable **VPC Flow Logs** for network traffic auditing
- Use **NAT Gateway** for outbound internet from private subnets

### Data Protection
- Enable **RDS encryption** at rest (AES-256) and enforce SSL connections
- Store all secrets in **AWS Secrets Manager** with automatic rotation
- Enable **ECR image scanning on push** to detect OS/package vulnerabilities
- Use **ECR image tag immutability** in production

### Auditing
- Enable **CloudTrail** for full API call audit logs across all services
- Use **CloudWatch Container Insights** for ECS/EKS metrics and logs

---

## Summary

| Task | Service | Status |
|---|---|---|
| Task 1 | ECR | Image built, tested & pushed to ECR |
| Task 2 | ECS | App deployed, accessible via ALB and EC2 IP |
| Task 3 | EKS | Kubernetes deployment using same ECR image |
| Task 4 | RDS | Database integrated with Secrets Manager injection |

### Verified Endpoints

| Access Method | URL | Response |
|---|---|---|
| EC2 Direct | `http://34.227.65.182` | Hello World! |
| ALB DNS | `http://balanceecs-2062376782.us-east-1.elb.amazonaws.com` | Hello World! |
