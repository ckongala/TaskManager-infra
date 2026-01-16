<div align="center">

# 🏗️ TaskManager Infrastructure

### *Terraform + Jenkins + EKS Infrastructure for Task Manager Application*

[![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)](https://terraform.io)
[![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com)
[![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)](https://jenkins.io)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io)
[![EKS](https://img.shields.io/badge/Amazon_EKS-FF9900?style=for-the-badge&logo=amazoneks&logoColor=white)](https://aws.amazon.com/eks/)

---

**Infrastructure as Code repository for provisioning Jenkins CI/CD server and Amazon EKS cluster to deploy the Task Manager application.**

[Architecture](#-architecture) •
[Related Project](#-related-project) •
[Quick Start](#-quick-start) •
[Infrastructure](#-infrastructure-components)

</div>

---

## 🔗 Related Project

<div align="center">

```
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                              │
│                         🔄 TWO REPOSITORIES WORKING TOGETHER                                │
│                                                                                              │
├─────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                              │
│   ┌─────────────────────────────────┐         ┌─────────────────────────────────┐          │
│   │                                 │         │                                 │          │
│   │   📦 TaskManager-infra          │         │   📱 TaskManager-Kubernetes-    │          │
│   │      (THIS REPO)                │         │      Driven-CI-CD               │          │
│   │                                 │         │                                 │          │
│   │   🏗️ INFRASTRUCTURE             │ ──────► │   💻 APPLICATION                 │          │
│   │                                 │ deploys │                                 │          │
│   │   • Jenkins Server (Terraform)  │         │   • Flask API (Python)          │          │
│   │   • EKS Cluster (Terraform)     │         │   • MySQL Database              │          │
│   │   • VPC & Networking            │         │   • Docker Configuration        │          │
│   │   • Jenkins Pipeline            │         │   • Kubernetes Manifests        │          │
│   │   • K8s Deployment Manifests    │         │   • GitHub Actions CI/CD        │          │
│   │                                 │         │   • Pytest Tests                │          │
│   └─────────────────────────────────┘         └─────────────────────────────────┘          │
│                                                                                              │
│                                    THE CONNECTION                                           │
│   ─────────────────────────────────────────────────────────────────────────────────────    │
│                                                                                              │
│   This repo provisions the AWS infrastructure (Jenkins + EKS), then Jenkins                │
│   pipeline deploys the Task Manager application from the other repository.                 │
│                                                                                              │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```

</div>

### 📌 Repository Links

| Repository | Description | Link |
|:-----------|:------------|:-----|
| **TaskManager-infra** | Infrastructure provisioning (Terraform, Jenkins, EKS) | 📍 *You are here* |
| **TaskManager-Kubernetes-Driven-CI-CD** | Flask application, Docker, K8s manifests | [View Repository →](../TaskManager-Kubernetes-Driven-CI-CD/) |

---

## 📑 Table of Contents

- [Related Project](#-related-project)
- [Architecture](#-architecture)
- [How They Connect](#-how-they-connect)
- [Infrastructure Components](#-infrastructure-components)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Project Structure](#-project-structure)
- [Jenkins Pipeline](#-jenkins-pipeline)
- [Terraform Commands](#-terraform-commands)

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                                      AWS CLOUD                                               │
├─────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                              │
│   ┌───────────────────────────────────────────────────────────────────────────────────┐     │
│   │                           VPC (Jenkins Server)                                     │     │
│   │                                                                                    │     │
│   │   ┌────────────────────────────────────────────────────────────────────────────┐  │     │
│   │   │                            Subnet                                           │  │     │
│   │   │                                                                             │  │     │
│   │   │   ┌─────────────────────────────────────────────────────────────────────┐  │  │     │
│   │   │   │                     EC2 Instance                                     │  │  │     │
│   │   │   │                                                                      │  │  │     │
│   │   │   │   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │  │  │     │
│   │   │   │   │   Jenkins    │  │  Terraform   │  │   kubectl    │             │  │  │     │
│   │   │   │   │   :8080      │  │              │  │              │             │  │  │     │
│   │   │   │   └──────────────┘  └──────────────┘  └──────────────┘             │  │  │     │
│   │   │   │                                                                      │  │  │     │
│   │   │   └─────────────────────────────────────────────────────────────────────┘  │  │     │
│   │   │                                                                             │  │     │
│   │   └────────────────────────────────────────────────────────────────────────────┘  │     │
│   │                                                                                    │     │
│   └───────────────────────────────────────────────────────────────────────────────────┘     │
│                                          │                                                   │
│                                          │ Jenkins Pipeline                                  │
│                                          │ creates & deploys to                              │
│                                          ▼                                                   │
│   ┌───────────────────────────────────────────────────────────────────────────────────┐     │
│   │                              VPC (EKS Cluster)                                     │     │
│   │                                                                                    │     │
│   │   ┌──────────────────────────────────────────────────────────────────────────┐    │     │
│   │   │                         EKS Cluster (K8s 1.29)                           │    │     │
│   │   │                                                                          │    │     │
│   │   │   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         │    │     │
│   │   │   │   Node Group    │  │   Node Group    │  │   (Auto-scale)  │         │    │     │
│   │   │   │   (t3.medium)   │  │   (t3.medium)   │  │   min:1 max:3   │         │    │     │
│   │   │   │                 │  │                 │  │                 │         │    │     │
│   │   │   │  ┌───────────┐  │  │  ┌───────────┐  │  │                 │         │    │     │
│   │   │   │  │Flask Pod  │  │  │  │MySQL Pod  │  │  │                 │         │    │     │
│   │   │   │  │           │  │  │  │           │  │  │                 │         │    │     │
│   │   │   │  └───────────┘  │  │  └───────────┘  │  │                 │         │    │     │
│   │   │   └─────────────────┘  └─────────────────┘  └─────────────────┘         │    │     │
│   │   │                                                                          │    │     │
│   │   └──────────────────────────────────────────────────────────────────────────┘    │     │
│   │                                                                                    │     │
│   └───────────────────────────────────────────────────────────────────────────────────┘     │
│                                                                                              │
│   ┌───────────────────────────────────────────────────────────────────────────────────┐     │
│   │                                    S3 Bucket                                       │     │
│   │                        (Terraform State Backend)                                   │     │
│   │                  jenkins-terraform-kubernetes-flaskapp-2024-v2                    │     │
│   └───────────────────────────────────────────────────────────────────────────────────┘     │
│                                                                                              │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 How They Connect

### The Complete CI/CD Flow

```
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                              END-TO-END DEPLOYMENT FLOW                                     │
├─────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                              │
│   STEP 1: Provision Jenkins Server                                                          │
│   ────────────────────────────────                                                          │
│                                                                                              │
│   Developer                                                                                  │
│       │                                                                                      │
│       │  terraform apply                                                                     │
│       ▼                                                                                      │
│   ┌─────────────────────┐                     ┌─────────────────────┐                       │
│   │  TaskManager-infra  │ ──────────────────► │   Jenkins Server    │                       │
│   │  (Terraform Code)   │    provisions       │   (EC2 Instance)    │                       │
│   └─────────────────────┘                     └──────────┬──────────┘                       │
│                                                          │                                   │
│   STEP 2: Jenkins Creates EKS Cluster                    │                                   │
│   ───────────────────────────────────                    │                                   │
│                                                          │                                   │
│                                                          │ runs Jenkinsfile                  │
│                                                          ▼                                   │
│   ┌─────────────────────┐                     ┌─────────────────────┐                       │
│   │  TaskManager-infra  │ ◄─────────────────  │   Jenkins Pipeline  │                       │
│   │  /eks-cluster/      │   terraform apply   │   Stage 1: Create   │                       │
│   │  (EKS Terraform)    │                     │   EKS Cluster       │                       │
│   └─────────────────────┘                     └──────────┬──────────┘                       │
│           │                                              │                                   │
│           │ creates                                      │                                   │
│           ▼                                              │                                   │
│   ┌─────────────────────┐                                │                                   │
│   │    EKS Cluster      │                                │                                   │
│   │   (Kubernetes)      │                                │                                   │
│   └──────────┬──────────┘                                │                                   │
│              │                                           │                                   │
│   STEP 3: Deploy Application                             │                                   │
│   ──────────────────────                                 │                                   │
│              │                                           │                                   │
│              │ kubectl apply            ┌────────────────┴───────────────┐                  │
│              │                          │   Jenkins Pipeline             │                  │
│              ▼                          │   Stage 2: Deploy to EKS       │                  │
│   ┌─────────────────────┐               │                                │                  │
│   │  TaskManager-       │ ◄─────────────│   • kubectl apply dep_db_pv    │                  │
│   │  Kubernetes-        │               │   • kubectl apply dep_db       │                  │
│   │  Driven-CI-CD       │               │   • kubectl apply dep_app      │                  │
│   │  (K8s Manifests)    │               │   • kubectl apply dep_test     │                  │
│   └─────────────────────┘               └────────────────────────────────┘                  │
│                                                                                              │
│   RESULT: Task Manager App Running on EKS! ✅                                               │
│                                                                                              │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```

### What Each Repository Provides

<table>
<tr>
<th width="50%">🏗️ TaskManager-infra (This Repo)</th>
<th width="50%">📱 TaskManager-Kubernetes-Driven-CI-CD</th>
</tr>
<tr>
<td>

**Infrastructure Provisioning:**
- ✅ Jenkins Server (EC2)
- ✅ EKS Cluster
- ✅ VPC & Networking
- ✅ Security Groups
- ✅ IAM Roles
- ✅ S3 State Backend

**CI/CD Orchestration:**
- ✅ Jenkinsfile Pipeline
- ✅ Jenkins Installation Script
- ✅ Kubernetes Manifests (copy)

</td>
<td>

**Application Code:**
- ✅ Flask REST API
- ✅ SQLAlchemy Models
- ✅ HTML/CSS Frontend

**Containerization:**
- ✅ Dockerfile
- ✅ Docker Compose

**CI/CD:**
- ✅ GitHub Actions Workflow
- ✅ Kubernetes Manifests
- ✅ Pytest Tests

</td>
</tr>
</table>

---

## 🧩 Infrastructure Components

### 1. Jenkins Server Infrastructure

| Resource | Description |
|:---------|:------------|
| **VPC** | Isolated network for Jenkins |
| **Subnet** | Public subnet with IGW access |
| **Internet Gateway** | Internet connectivity |
| **Route Table** | Routes to IGW |
| **Security Group** | SSH (22) + Jenkins (8080) |
| **EC2 Instance** | Amazon Linux 2 with Jenkins |

### 2. EKS Cluster Infrastructure

| Resource | Description |
|:---------|:------------|
| **VPC Module** | Production-grade VPC |
| **EKS Module** | Managed Kubernetes cluster |
| **Node Group** | t3.medium instances (1-3 nodes) |
| **Kubernetes** | Version 1.29 |

### 3. Installed Tools on Jenkins Server

```bash
# Automatically installed via user_data script:
├── Jenkins          # CI/CD automation server
├── Java 17          # Jenkins runtime
├── Git              # Version control
├── Terraform        # Infrastructure as Code
└── kubectl          # Kubernetes CLI
```

---

## 📋 Prerequisites

- [Terraform](https://terraform.io/downloads) >= 1.0
- [AWS CLI](https://aws.amazon.com/cli/) configured
- AWS Account with admin permissions
- S3 bucket for Terraform state
- EC2 Key Pair named `project_keypair`

---

## 🚀 Quick Start

### Step 1: Clone Repository

```bash
git clone <repository-url>
cd TaskManager-infra
```

### Step 2: Create S3 Backend Bucket

```bash
aws s3 mb s3://jenkins-terraform-kubernetes-flaskapp-2024-v2 --region ap-northeast-1
```

### Step 3: Create EC2 Key Pair

```bash
aws ec2 create-key-pair --key-name project_keypair --region ap-northeast-1 \
  --query 'KeyMaterial' --output text > project_keypair.pem
chmod 400 project_keypair.pem
```

### Step 4: Create terraform.tfvars

```hcl
vpc_cidr_block    = "10.0.0.0/16"
subnet_cidr_block = "10.0.1.0/24"
availability_zone = "ap-northeast-1a"
env_prefix        = "dev"
instance_type     = "t2.medium"
```

### Step 5: Deploy Jenkins Server

```bash
# Initialize Terraform
terraform init

# Preview changes
terraform plan

# Apply infrastructure
terraform apply
```

### Step 6: Access Jenkins

```bash
# Get Jenkins URL
terraform output ec2_public_ip

# Access Jenkins at: http://<ec2_public_ip>:8080

# SSH to get initial password
ssh -i project_keypair.pem ec2-user@<ec2_public_ip>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Step 7: Configure Jenkins Pipeline

1. Add AWS credentials in Jenkins
2. Create pipeline job pointing to this repo
3. Run the pipeline to create EKS and deploy app

---

## 📁 Project Structure

```
TaskManager-infra/
│
├── 📄 provider.tf            # AWS provider configuration
├── 📄 backend.tf             # S3 backend for state
├── 📄 variables.tf           # Input variables
├── 📄 outputs.tf             # Output values
│
├── 📄 vpc.tf                 # VPC and Subnet
├── 📄 route.tf               # Internet Gateway + Route Table
├── 📄 security.tf            # Security Group
├── 📄 server.tf              # EC2 Instance (Jenkins)
│
├── 📄 jenkins-script.sh      # User data script (Jenkins install)
├── 📄 Jenkinsfile            # CI/CD Pipeline definition
│
├── 📁 eks-cluster/           # EKS Cluster Terraform
│   ├── 📄 vpc.tf             # EKS VPC module
│   ├── 📄 eks.tf             # EKS cluster module
│   ├── 📄 variables.tf       # EKS variables
│   ├── 📄 terraform.tfvars   # EKS variable values
│   └── 📄 backend.tf         # EKS state backend
│
├── 📁 kubernetes/            # K8s Manifests (deployed by Jenkins)
│   ├── 📄 dep_app.yaml       # Flask application
│   ├── 📄 dep_db.yaml        # MySQL database
│   ├── 📄 dep_db_pv.yaml     # Persistent Volume
│   └── 📄 dep_test.yaml      # Pytest deployment
│
├── 📄 .gitignore
└── 📄 README.md
```

---

## 🔧 Jenkins Pipeline

### Pipeline Stages

```groovy
pipeline {
    agent any
    
    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = "ap-northeast-1"
    }
    
    stages {
        stage("Create an EKS Cluster") {
            // terraform init && terraform apply
            // Creates EKS cluster using eks-cluster/ terraform
        }
        
        stage("Deploy to EKS") {
            // kubectl apply -f kubernetes/*.yaml
            // Deploys Flask app, MySQL, and tests
        }
    }
}
```

### Pipeline Visualization

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          JENKINS PIPELINE                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────┐      ┌─────────────────────────────┐     │
│   │                             │      │                             │     │
│   │   Stage 1: Create EKS      │ ───► │   Stage 2: Deploy to EKS   │     │
│   │                             │      │                             │     │
│   │   cd eks-cluster/           │      │   aws eks update-kubeconfig │     │
│   │   terraform init            │      │   kubectl apply dep_db_pv   │     │
│   │   terraform apply           │      │   kubectl apply dep_db      │     │
│   │                             │      │   kubectl apply dep_app     │     │
│   │   ⏱️ ~15-20 minutes          │      │   kubectl apply dep_test    │     │
│   │                             │      │                             │     │
│   └─────────────────────────────┘      └─────────────────────────────┘     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Required Jenkins Credentials

| Credential ID | Type | Description |
|:--------------|:-----|:------------|
| `AWS_ACCESS_KEY_ID` | Secret text | AWS access key |
| `AWS_SECRET_ACCESS_KEY` | Secret text | AWS secret key |

---

## 📋 Terraform Commands

### Jenkins Server

```bash
cd TaskManager-infra/

# Initialize
terraform init

# Plan
terraform plan -var-file="terraform.tfvars"

# Apply
terraform apply -var-file="terraform.tfvars"

# Destroy
terraform destroy -var-file="terraform.tfvars"
```

### EKS Cluster

```bash
cd TaskManager-infra/eks-cluster/

# Initialize
terraform init

# Plan
terraform plan

# Apply
terraform apply

# Destroy
terraform destroy
```

---

## ⚙️ Configuration

### Variables (Jenkins Server)

| Variable | Description | Example |
|:---------|:------------|:--------|
| `vpc_cidr_block` | VPC CIDR | `10.0.0.0/16` |
| `subnet_cidr_block` | Subnet CIDR | `10.0.1.0/24` |
| `availability_zone` | AWS AZ | `ap-northeast-1a` |
| `env_prefix` | Environment prefix | `dev` |
| `instance_type` | EC2 type | `t2.medium` |

### Variables (EKS Cluster)

| Variable | Description | Example |
|:---------|:------------|:--------|
| `vpc_cidr_block` | VPC CIDR | `10.0.0.0/16` |
| `public_subnet_cidr_blocks` | Public subnets | `["10.0.1.0/24", "10.0.2.0/24"]` |
| `private_subnet_cidr_blocks` | Private subnets | `["10.0.3.0/24", "10.0.4.0/24"]` |

---

## 🔐 Security Notes

- 🔒 Security group allows SSH (22) and Jenkins (8080) from anywhere
- 🔒 Restrict SSH access to your IP in production
- 🔒 Store Terraform state in S3 with encryption
- 🔒 Use IAM roles instead of access keys when possible
- 🔒 Enable Jenkins security and authentication

---

## 💰 Cost Estimation

| Resource | Estimated Monthly Cost |
|:---------|:----------------------|
| EC2 (t2.medium - Jenkins) | ~$35-40 |
| EKS Cluster | ~$72 |
| EC2 Node Group (2x t3.medium) | ~$60-70 |
| NAT Gateway (if enabled) | ~$32 |
| S3 (State) | < $1 |
| **Total** | **~$170-220/month** |

> 💡 Destroy resources when not in use: `terraform destroy`

---

## 🔗 Quick Links

| Resource | Link |
|:---------|:-----|
| 📱 **Application Repo** | [TaskManager-Kubernetes-Driven-CI-CD](../TaskManager-Kubernetes-Driven-CI-CD/) |
| 📚 **AWS EKS Docs** | [aws.amazon.com/eks](https://aws.amazon.com/eks/) |
| 📚 **Terraform EKS Module** | [registry.terraform.io](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws) |
| 📚 **Jenkins Docs** | [jenkins.io](https://www.jenkins.io/doc/) |

---

<div align="center">

## 🔄 Remember the Connection!

**This repo provisions the infrastructure.**
**The [TaskManager-Kubernetes-Driven-CI-CD](../TaskManager-Kubernetes-Driven-CI-CD/) repo contains the application.**

```
TaskManager-infra (Infrastructure) ──deploys──► TaskManager-Kubernetes-Driven-CI-CD (App)
```

---

**Built with ❤️ using Terraform, Jenkins, and AWS EKS**

⭐ Star this repo if you find it useful!

</div>

