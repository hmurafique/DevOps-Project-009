# 🚀 DevOps Project 009
# Deploy an E-Commerce Three-Tier Application on AWS EKS with Helm

<p align="center">
  <img src="https://imgur.com/placeholder-robotshop.png" alt="RobotShop EKS Banner" width="800"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/AWS_EKS-FF9900?style=flat&logo=amazonaws&logoColor=white" alt="AWS EKS"/>
  <img src="https://img.shields.io/badge/Kubernetes-1.34-326CE5?style=flat&logo=kubernetes&logoColor=white" alt="Kubernetes"/>
  <img src="https://img.shields.io/badge/Helm-3.21.0-0F1689?style=flat&logo=helm&logoColor=white" alt="Helm"/>
  <img src="https://img.shields.io/badge/eksctl-0.227.0-FF9900?style=flat&logo=amazonaws&logoColor=white" alt="eksctl"/>
  <img src="https://img.shields.io/badge/AWS_ALB-FF9900?style=flat&logo=amazonaws&logoColor=white" alt="ALB"/>
  <img src="https://img.shields.io/badge/AWS_EBS-FF9900?style=flat&logo=amazonaws&logoColor=white" alt="EBS"/>
  <img src="https://img.shields.io/badge/kubectl-1.36.1-326CE5?style=flat&logo=kubernetes&logoColor=white" alt="kubectl"/>
  <img src="https://img.shields.io/badge/Ubuntu-22.04-E95420?style=flat&logo=ubuntu&logoColor=white" alt="Ubuntu"/>
</p>

---

## 📌 Project Overview

This project demonstrates deploying a fully functional **3-Tier E-Commerce Application (Stan's Robot Shop)** on **AWS EKS** using **Helm**. The application consists of 12 microservices built with different technologies including NodeJS, Java, Python, Golang, PHP, MongoDB, Redis, MySQL, and RabbitMQ — all orchestrated on Kubernetes with persistent storage and internet-facing load balancing via AWS ALB.

### ✅ Key Features

- **EKS Cluster** — Managed Kubernetes on AWS with t3.medium worker nodes
- **Helm Deployment** — Single command to deploy all 12 microservices
- **AWS ALB Ingress** — Internet-facing Application Load Balancer
- **EBS CSI Driver** — Persistent storage for stateful services (Redis, MySQL, MongoDB)
- **IAM OIDC** — Secure pod-level AWS permissions via IAM roles
- **Auto Scaling** — Node group with min/max scaling configured

---

## 🏗️ Architecture

```
Internet
    │
    ▼
AWS ALB (Application Load Balancer)
    │  [internet-facing, port 80]
    ▼
Kubernetes Ingress (robot-shop namespace)
    │
    ▼
┌────────────────────────────────────────────┐
│           AWS EKS Cluster                  │
│         (Kubernetes v1.34)                 │
│                                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │   web    │  │ catalogue│  │   cart   │  │
│  │ (nginx)  │  │ (NodeJS) │  │ (NodeJS) │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│                                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  user    │  │ payment  │  │ shipping │  │
│  │ (NodeJS) │  │ (Python) │  │  (Java)  │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│                                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ ratings  │  │dispatch  │  │rabbitmq  │  │
│  │  (PHP)   │  │ (Golang) │  │          │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│                                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ mongodb  │  │  mysql   │  │  redis   │  │
│  │  (EBS)   │  │  (EBS)   │  │  (EBS)   │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│                                            │
│  Node Group: 2x t3.medium (us-east-1)      │
└────────────────────────────────────────────┘
         │                    │
    EBS Volume           EBS Volume
   (Persistent)         (Persistent)
```

---

## 🛠️ Tools & Versions

| Tool | Version | Purpose |
|------|---------|---------|
| AWS EKS | Kubernetes 1.34 | Managed Kubernetes |
| eksctl | 0.227.0 | EKS Cluster Management |
| kubectl | 1.36.1 | Kubernetes CLI |
| Helm | 3.21.0 | K8s Package Manager |
| AWS CLI | 2.34.64 | AWS Management |
| AWS ALB Controller | 2.11.0 | Load Balancer |
| EBS CSI Driver | Latest | Persistent Storage |
| EC2 Jump Server | t2.micro Ubuntu 22.04 | Management Node |
| Worker Nodes | t3.medium x2 | Compute |

---

## 📁 Repository Structure

```
DevOps-Project-009/
├── K8s/
│   └── helm/               # Helm chart for RobotShop
│       ├── values.yaml     # Modified: gp2 StorageClass
│       └── templates/      # K8s manifests
├── app/                    # Microservices source code
│   ├── cart/               # NodeJS
│   ├── catalogue/          # NodeJS
│   ├── dispatch/           # Golang
│   ├── mongodb/            # MongoDB
│   ├── mysql/              # MySQL
│   ├── payment/            # Python
│   ├── ratings/            # PHP
│   ├── shipping/           # Java
│   ├── user/               # NodeJS
│   └── web/                # Nginx + AngularJS
├── ingress.yaml            # AWS ALB Ingress config
├── iam_policy.json         # ALB Controller IAM policy
└── README.md
```

---

## 🚀 Step-by-Step Implementation Guide

### Prerequisites

- AWS Account with IAM user (AdministratorAccess)
- EC2 Key Pair
- Basic knowledge of Kubernetes

---

### PHASE 1 — Jump Server (EC2) Setup

#### 1.1 — Launch EC2 Instance

| Setting | Value |
|---------|-------|
| Name | `eks-server` |
| AMI | Ubuntu 22.04 LTS |
| Instance Type | `t2.micro` |
| Storage | 20 GB |
| Security Group | Port 22 (SSH) only |

#### 1.2 — Install Required Tools

```bash
# System update
sudo apt update && sudo apt upgrade -y

# AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install -y unzip
unzip awscliv2.zip
sudo ./aws/install
rm -rf awscliv2.zip aws/
aws --version

# kubectl (latest stable)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo install -m 0755 kubectl /usr/local/bin/kubectl
rm kubectl
kubectl version --client

# eksctl (latest)
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
eksctl version

# Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

#### 1.3 — AWS Configure

```bash
aws configure
# Enter: Access Key, Secret Key, Region (us-east-1), Output (json)

# Verify
aws sts get-caller-identity
```

---

### PHASE 2 — EKS Cluster Setup

#### 2.1 — Create EKS Cluster

```bash
eksctl create cluster \
  --name robot-shop-cluster \
  --region us-east-1 \
  --nodegroup-name robot-shop-ng \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3
```

⚠️ Takes 15-20 minutes

```bash
# Verify nodes
kubectl get nodes
```

#### 2.2 — Configure IAM OIDC Provider

```bash
export cluster_name=robot-shop-cluster
export account_id=$(aws sts get-caller-identity --query Account --output text)

# Get OIDC ID
oidc_id=$(aws eks describe-cluster --name $cluster_name \
  --query "cluster.identity.oidc.issuer" \
  --output text | cut -d '/' -f 5)

# Check if already exists
aws iam list-open-id-connect-providers | grep $oidc_id

# Associate OIDC provider (if not exists)
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

---

### PHASE 3 — AWS ALB Controller Setup

#### 3.1 — Create IAM Policy

```bash
# Download latest policy (v2.11.0)
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

#### 3.2 — Create IAM Service Account

```bash
eksctl create iamserviceaccount \
  --cluster=$cluster_name \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::${account_id}:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

#### 3.3 — Install ALB Controller via Helm

```bash
# Get VPC ID
export vpc_id=$(aws eks describe-cluster \
  --name $cluster_name \
  --query "cluster.resourcesVpcConfig.vpcId" \
  --output text)

helm repo add eks https://aws.github.io/eks-charts
helm repo update eks

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=$cluster_name \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=$vpc_id

# Verify
kubectl get deployment -n kube-system aws-load-balancer-controller
```

---

### PHASE 4 — EBS CSI Driver Setup

```bash
# Create IAM Role
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster $cluster_name \
  --role-name AmazonEKS_EBS_CSI_DriverRole \
  --role-only \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve

# Install EBS CSI Driver addon
eksctl create addon \
  --name aws-ebs-csi-driver \
  --cluster $cluster_name \
  --service-account-role-arn arn:aws:iam::${account_id}:role/AmazonEKS_EBS_CSI_DriverRole \
  --force

# Verify
kubectl get pods -n kube-system | grep ebs
```

---

### PHASE 5 — Deploy RobotShop with Helm

```bash
# Clone RobotShop repo
git clone https://github.com/instana/robot-shop.git
cd robot-shop

# Fix StorageClass (EKS uses gp2, not standard)
sed -i 's/storageClassName: standard/storageClassName: gp2/' K8s/helm/values.yaml

# Create namespace
kubectl create namespace robot-shop

# Deploy with Helm
helm install robot-shop K8s/helm \
  --namespace robot-shop

# Verify pods (takes 2-3 minutes)
kubectl get pods -n robot-shop
```

---

### PHASE 6 — Configure Ingress (AWS ALB)

```bash
kubectl apply -f ingress.yaml

# Wait for ALB to provision (2-3 minutes)
kubectl get ingress -n robot-shop
```

Access the app:
```
http://<ALB-ADDRESS>
```

---

## 📊 Deployed Microservices

| Service | Technology | Port |
|---------|-----------|------|
| web | Nginx + AngularJS | 8080 |
| catalogue | NodeJS | 8080 |
| cart | NodeJS | 8080 |
| user | NodeJS | 8080 |
| payment | Python | 8080 |
| shipping | Java (Spring Boot) | 8080 |
| ratings | PHP (Apache) | 80 |
| dispatch | Golang | 55555 |
| mongodb | MongoDB | 27017 |
| mysql | MySQL 5.7 | 3306 |
| redis | Redis | 6379 |
| rabbitmq | RabbitMQ | 5672 |

---

## ⚠️ Common Issues & Fixes

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| Pods `Pending` — Insufficient memory | t3.small not enough RAM | Use t3.medium node group |
| `data-redis-0` PVC Pending | `standard` StorageClass not in EKS | Change to `gp2` in values.yaml |
| ALB ADDRESS empty | IAM policy missing `DescribeListenerAttributes` | Update IAM policy to v2.11.0 |
| `Backend service does not exist` | web service not deployed | Run `helm upgrade` to create web service |
| StatefulSet immutable error | Can't patch StorageClass on running StatefulSet | Helm uninstall → fix values.yaml → reinstall |
| Shipping pod `0/1` | MySQL connection refused on startup | `kubectl rollout restart deployment/shipping` |

---

## 💰 Cost Optimization

Scale down nodes when not in use:

```bash
# Scale down to 0 (stop billing for EC2 nodes)
eksctl scale nodegroup \
  --cluster robot-shop-cluster \
  --name robot-shop-ng-medium \
  --nodes 0 \
  --nodes-min 0 \
  --nodes-max 3

# Scale back up
eksctl scale nodegroup \
  --cluster robot-shop-cluster \
  --name robot-shop-ng-medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3
```

---

## 🧹 Cleanup

```bash
# Delete everything (EKS cluster, nodes, VPC, etc.)
eksctl delete cluster --name robot-shop-cluster --region us-east-1
```

⚠️ This will delete all resources including ALB, node groups, and VPC.

---

## 📚 Reference

- [Original Project Reference](https://github.com/NotHarshhaa/DevOps-Projects/tree/master/DevOps-Project-15)
- [RobotShop by Instana](https://github.com/instana/robot-shop)
- [eksctl Documentation](https://eksctl.io)
- [AWS ALB Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller)
- [EBS CSI Driver](https://github.com/kubernetes-sigs/aws-ebs-csi-driver)
- [Helm Documentation](https://helm.sh/docs)

---

## 🛠️ Author

**Hafiz Muhammad Umar Rafique**
CloudOps & DevOps Engineer

[![GitHub](https://img.shields.io/badge/GitHub-hmurafique-181717?style=flat&logo=github&logoColor=white)](https://github.com/hmurafique)
[![DockerHub](https://img.shields.io/badge/DockerHub-hmurafique93-2496ED?style=flat&logo=docker&logoColor=white)](https://hub.docker.com/u/hmurafique93)

---

> **Note:** This project is part of a hands-on DevOps learning series based on [NotHarshhaa/DevOps-Projects](https://github.com/NotHarshhaa/DevOps-Projects). All implementation, fixes, and configurations are done independently from scratch.
