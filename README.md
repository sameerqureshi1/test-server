# Deploying CV Website with GitHub Self-Hosted Runner on AWS EC2

This repository (`test-server`) demonstrates how to deploy a simple **CV website** to an **AWS EC2 instance** using a **GitHub self-hosted runner**. The deployment is automated with GitHub Actions, Docker, and Nginx.
---

## 🚀 Objective

Automate the deployment of a CV (`index.html`) hosted in Docker on an AWS EC2 instance whenever changes are pushed to the repository.

---

## 📋 Prerequisites

Before proceeding, ensure you have:

- A **GitHub account** and repository `test-server`
- An **AWS EC2 instance** (Ubuntu or similar) with **SSH access**
- **Docker** installed on the EC2 instance (or Docker Engine + docker-compose)
- A **Git client** on the EC2 instance
- A **GitHub self-hosted runner** configured on the EC2 instance
- **EC2 Security Group** allowing required ports:
  - `22` → SSH
  - `80/443` → HTTP/HTTPS

---

## 📂 Repository Structure

Place the following files in the repository root:

- `Dockerfile` → builds the image that serves `index.html`
- `index.html` → your CV file
- `.github/workflows/ci.yml` → GitHub Actions workflow targeting the self-hosted runner

---

## 🛠 Step-by-Step Deployment Guide

### 1. Create GitHub Repository
On GitHub, create a new repository named **`test-server`**.
<img width="1342" height="609" alt="step 1" src="https://github.com/user-attachments/assets/5855e81e-7e0f-4100-bc33-f19c5b8e06ee" />

---

### 2. Clone Repository on EC2
Run the following on your EC2 VM:

git clone https://github.com/<your-username>/test-server.git
cd test-server
<img width="1350" height="646" alt="Step 2 " src="https://github.com/user-attachments/assets/767ebed4-88ce-4e6d-8740-32c5518d7e4f" />

### 3. Verify Project Files
Check if project files are present:

ls -la
### You should see Dockerfile, index.html, and .github/workflows/
<img width="1360" height="703" alt="step 4" src="https://github.com/user-attachments/assets/b79b2e2e-f4dc-4f30-9281-05483b5534fc" />

### 4. Create Dockerfile (example using Nginx)


FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80


<img width="1352" height="663" alt="Dockerfile" src="https://github.com/user-attachments/assets/38dfd2ad-7579-4dbb-8e2c-30dea694fae7" />



### 5. Install & Configure GitHub Self-Hosted Runner
1. Go to **GitHub Repository → Settings → Actions → Runners → Add runner**
2. Select **Linux** and copy the commands shown.
3. Run those commands on your EC2 instance:
<img width="1065" height="496" alt="image" src="https://github.com/user-attachments/assets/e3d2ad0f-48bf-432c-9867-e7fd5731cab4" />
<img width="1078" height="626" alt="image" src="https://github.com/user-attachments/assets/cbfdd3f4-f371-4f56-bede-2537c1485113" />
<img width="1054" height="630" alt="image" src="https://github.com/user-attachments/assets/c139c8e5-a7d8-472d-8d1b-52717e3a4e2d" />

### 6. Create GitHub Actions Workflow
name: Build, Deploy & Test CV

on:
  push:
    branches:
      - main

jobs:
  build-deploy-test:
    runs-on: self-hosted

    steps:
      # Step 1: Checkout repository
      - name: Checkout code
        uses: actions/checkout@v3

      # Step 2: Build Docker image
      - name: Build Docker image
        run: docker build -t my-cv:latest .

      # Step 3: Stop & remove old container
      - name: Stop old container
        run: |
          if [ "$(docker ps -q -f name=my-cv-container)" ]; then
            docker stop my-cv-container
            docker rm my-cv-container
          fi

      # Step 4: Run new container
      - name: Run container
        run: docker run -d -p 8083:80 --name my-cv-container my-cv:latest

      # Step 5: Test deployment
      - name: Test if CV is served
        run: |
          sleep 5
          if curl -s http://localhost:8083 | grep -q "CV"; then
            echo "✅ Test Passed: CV is served correctly"
          else
            echo "❌ Test Failed: CV not found"
            echo "Page content was:"
            curl -s http://localhost:8083
            exit 1
          fi

<img width="1345" height="650" alt="ci yml" src="https://github.com/user-attachments/assets/215ad677-81cf-4d18-9608-3ffc1d755934" />


### 7. Push Changes & Trigger Workflow
Commit and push the workflow to `main` (or default) branch. On push, GitHub Actions will queue the job. Since `runs-on: self-hosted` the job will be picked up by your EC2 runner.
<img width="1054" height="512" alt="image" src="https://github.com/user-attachments/assets/ac7c7859-3599-4978-ad51-783e2ed9519f" />

### 8. Verify Service on Browser
On your local browser go to http://<EC2_PUBLIC_IP>/ to see the CV (index.html). Ensure Security Group allows port 80.
<img width="1343" height="683" alt="Self hosted runner on ec2 instance" src="https://github.com/user-attachments/assets/12cd5a52-9fa9-4236-930c-21d565c9654f" />
### Final Notes

Builds run directly on your EC2 VM

Keep your self-hosted runner updated

Monitor resource usage

Re-register runner if tokens expire

