# 🚀 Node.js DevOps Project - CI/CD to AWS EC2

This project demonstrates how to set up a full CI/CD pipeline using GitHub Actions to deploy a Node.js application on an AWS EC2 instance.


---

## 📘 Project Overview

- **Goal**: Automate deployment of a Node.js app to an EC2 instance
- **Tools**: GitHub Actions, AWS EC2, SSH, Node.js, npm, bash

---

## ☁️ EC2 Instance Setup

- **OS**: Ubuntu 24.04 LTS
- **Security Group Inbound Rules**:
  - `22` SSH: Your IP only
  - `80` HTTP: 0.0.0.0/0
  - `3000` Node.js App: 0.0.0.0/0

---

## 🔐 SSH Key Setup

1. **Generate SSH Key**:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "github-to-ec2"
   ```
2. **Save to**: `~/.ssh/github_deploy_key`
3. **Public Key** → EC2 `/home/ubuntu/.ssh/authorized_keys`
4. **Private Key** → GitHub Secret `EC2_SSH_KEY`

---

## 📁 Project Structure

```
my-devops-app/
├── Dockerfile
├── README.md
├── index.js
├── package.json
├── .github/
│   └── workflows/
│       └── deploy.yml
```

---

## ⚙️ GitHub Actions Workflow

- **Trigger**: On push to `main`
- **Location**: `.github/workflows/deploy.yml`

### 🔄 Workflow Summary:

1. Checkout the code
2. Inject SSH key
3. SSH into EC2 and:
   - Install nodejs, npm, git, lsof
   - Kill any process using port `3000`
   - Clone the repo
   - Run the Node.js app with `nohup`

```yaml
name: Deploy to EC2

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v3

      - name: Add SSH key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.EC2_SSH_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa

      - name: Deploy to EC2
        run: |
          ssh -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa ubuntu@<YOUR_EC2_PUBLIC_IP> << 'EOF'
            sudo apt update
            sudo apt install -y nodejs npm git lsof
            rm -rf my-devops-app
            git clone https://github.com/imajedkd/my-devops-app.git
            cd my-devops-app
            pid=$(sudo lsof -t -i:3000)
            if [ ! -z "$pid" ]; then
              echo "Port 3000 in use, killing process $pid"
              sudo kill -9 $pid
            else
              echo "Port 3000 is free"
            fi
            npm install
            nohup node index.js > output.log 2>&1 &
          EOF
```

---

## ✅ Final Output

Open your browser and go to:

```
http://<YOUR_EC2_PUBLIC_IP>:3000
```

Expected result:

```
Hello, DevOps World!
```

---

## 🧠 Notes

- Always secure your private key using GitHub Secrets
- Avoid exposing port 22 to the public
- Use `nohup` to run apps in background
- Re-run GitHub Action after every push to `main`

---

Happy deploying! 🌍🚀
