# 🚀 Automated Website Deployment using GitHub Actions, Docker & AWS EC2

---

## 📌 Project Overview

This project demonstrates a complete CI/CD pipeline where a website is automatically deployed to an AWS EC2 instance using **GitHub Actions** and **Docker**.
Whenever code is pushed to GitHub, GitHub Actions securely connects to the EC2 instance and deploys the latest version of the website.

This project represents a **real-world DevOps automation workflow** used in production environments.

---

## 🧱 Architecture

```
GitHub Push
   ↓
GitHub Actions (CI/CD)
   ↓
SSH into AWS EC2
   ↓
Docker Build & Run
   ↓
Website Live on EC2
```

---

## 🛠️ Technologies Used

* AWS EC2 (Amazon Linux 2, Free Tier)
* IAM (User, Access Key, Secret Key)
* GitHub Actions (CI/CD)
* Docker
* Nginx
* Linux

---

## 📁 Project Structure

```
aws-devops-project/
│
├── index.html
├── Dockerfile
└── .github/
    └── workflows/
        └── deploy.yml
```

---

## 🔐 IAM User & Security Configuration (IMPORTANT)

### Why IAM User is Required

GitHub Actions must securely access AWS resources.
Instead of using the root account, an **IAM user** is created with limited permissions.

This follows **AWS security best practices**.

---

### Step 1: Create IAM User

AWS Console → **IAM → Users → Create user**

* User name: `github-actions-user`
* Access type: **Programmatic access**
* Permissions:

  * `AmazonEC2FullAccess` (for learning/demo purposes)

---

### Step 2: Generate Access Keys

After creating the user, download:

* **Access Key ID**
* **Secret Access Key**

⚠️ These credentials are generated **only once**.

---

### Step 3: Store Credentials Securely in GitHub

GitHub Repo → **Settings → Secrets → Actions**

Add the following secrets:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

These keys allow GitHub Actions to authenticate with AWS **without hardcoding credentials**.

---

## 🖥️ EC2 Setup

### Launch EC2 Instance

* AMI: Amazon Linux 2
* Instance Type: `t2.micro` (Free Tier)
* Inbound Rules:

  * SSH (22) → 0.0.0.0/0
  * HTTP (80) → 0.0.0.0/0

---

### Install Docker on EC2

```bash
sudo yum update -y
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
sudo chmod 666 /var/run/docker.sock
```

---

### Install Git

```bash
sudo yum install git -y
```

---

## 🐳 Dockerfile

```dockerfile
FROM nginx:latest

RUN rm -rf /usr/share/nginx/html/*
COPY index.html /usr/share/nginx/html/

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 📄 Sample index.html

```html
<h1>Hello from DevOps CI/CD Project 🚀</h1>
<p>Deployed automatically using GitHub Actions, Docker, and AWS EC2</p>
```

---

## 🔑 GitHub Secrets Required

Add these secrets under:

```
GitHub → Settings → Secrets → Actions
```

| Secret Name             | Description                             |
| ----------------------- | --------------------------------------- |
| `EC2_HOST`              | EC2 Public IPv4 address                 |
| `EC2_USER`              | ec2-user                                |
| `EC2_KEY`               | Full content of EC2 private `.pem` file |
| `AWS_ACCESS_KEY_ID`     | IAM user access key                     |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret key                     |

---

## ⚙️ GitHub Actions Workflow

Create file:

```
.github/workflows/deploy.yml
```

```yaml
name: Deploy Website to EC2

on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Deploy to EC2 via SSH
      uses: appleboy/ssh-action@v1.0.0
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USER }}
        key: ${{ secrets.EC2_KEY }}
        script: |
          cd /home/ec2-user
          sudo yum install git -y

          if [ ! -d "aws-devops-project" ]; then
            git clone https://github.com/YOUR-USERNAME/YOUR-REPO.git
          fi

          cd YOUR-REPO
          git pull

          sudo yum install docker -y
          sudo systemctl start docker
          sudo chmod 666 /var/run/docker.sock

          docker stop website || true
          docker rm website || true
          docker rmi website || true

          docker build -t website .
          docker run -d -p 80:80 --name website website
```

---

## 🚀 Deployment Flow

1. Developer pushes code to GitHub
2. GitHub Actions pipeline triggers
3. Secure authentication using IAM credentials
4. SSH into EC2
5. Docker image is built and deployed
6. Website updates automatically

---

## 🌍 Access the Application

```
http://<EC2-PUBLIC-IP>
```

---

## 💰 Cost Considerations

* Uses AWS Free Tier (`t2.micro`)
* GitHub Actions free minutes
* No paid AWS services used

Stop EC2 when not in use to avoid charges.

---

## 🧠 Key Learnings

* IAM user & secure credential management
* GitHub Actions CI/CD pipelines
* Docker-based deployments
* AWS EC2 automation
* Production-style DevOps workflow

---

## 👨‍💻 Author

DevOps Project by **[Your Name]**

---
