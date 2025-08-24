🚀 Node.js CI/CD Pipeline with GitHub Actions & AWS EC2

This project demonstrates a production-grade CI/CD pipeline for a Node.js application using GitHub Actions and a self-hosted runner on AWS EC2.

The pipeline ensures reliable testing, artifact management, and automated deployment with PM2 process manager for zero-downtime server operation.

✨ Features

✅ Automated unit testing on every push & pull request

📦 Artifact management for test results & logs

☁️ Continuous Deployment to AWS EC2 (self-hosted runner)

🔄 Process management & monitoring with PM2

🔍 Built-in application health checks after deployment

🏗️ Pipeline Architecture

## flowchart TD

    A[Developer Push/PR] --> B[Test Job - GitHub Runner]
    B -->|Run Jest/Mocha Tests| C[Upload Test Results as Artifact]
    C --> D[Deploy Job - Self-hosted EC2 Runner]
    D -->|Download Artifact| E[Deploy with PM2]
    E --> F[Health Check - curl localhost:3000]
    F --> G[Running Application 🎉]

##  Test Job (GitHub-hosted Runner)

Checks out repository

Sets up Node.js environment

Installs dependencies

Runs tests & captures results

Uploads results as artifact

## Workflow File
workflow to ensure proper connection. Create/update .github/workflows/ci-cd-pipeline.yml:Deploy Job (EC2 Self-hosted Runner)

Downloads test artifact

Displays results for verification

Deploys Node.js app with PM2

Performs health check to ensure availability

## ⚙️ Setup Instructions
1. Clone Repository
git clone https://github.com/tahmidmir/nodejs-ci-cd-app.git
cd nodejs-ci-cd-app

## 2. Configure GitHub Actions Workflow

Create .github/workflows/ci-cd.yml with test & deploy jobs. Example structure:

name: Node.js CI/CD

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm test
      - uses: actions/upload-artifact@v3
        with:
          name: test-results
          path: test-results.txt

  deploy:
    runs-on: self-hosted
    needs: test
    steps:
      - uses: actions/download-artifact@v3
        with:
          name: test-results
          path: ./test-results
      - run: cat ./test-results/test-results.txt
      - run: pm2 restart src/server.js || pm2 start src/server.js --name nodejs-app
      - run: curl http://localhost:3000

## 3. Setup Self-Hosted Runner on AWS EC2
# Update & install dependencies
sudo apt update && sudo apt upgrade -y
sudo apt install nodejs npm git -y

# Install PM2
sudo npm install -g pm2

# Register GitHub self-hosted runner
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64.tar.gz -L https://github.com/actions/runner/releases/download/v2.x.x/actions-runner-linux-x64-x.x.x.tar.gz
tar xzf ./actions-runner-linux-x64.tar.gz

./config.sh --url https://github.com/tahmidmir/nodejs-ci-cd-app --token <ghp_GWM0NeWoKaqINmHv0HxqTqqS2eJabB49AeX>

sudo ./svc.sh install

sudo ./svc.sh start

## Step 4: Verify the Runner Connection
Check the runner status on GitHub:

Go to your repository → Settings → Actions → Runners

Verify your self-hosted runner appears and is "Online"

Check the runner status on your EC2 instance:

Step 5: Test the Connection
Push the changes to trigger the workflow:

## bash
git add .github/workflows/ci-cd-pipeline.yml
git commit -m "Update workflow for EC2 connection"
git push origin main

## Step 7: Verify Successful Deployment
Once the workflow completes successfully:

SSH into your EC2 instance:

## bash
ssh -i your-key.pem ubuntu@your-ec2-public-ip
Check if the application is running:

## bash
pm2 status
curl http://localhost:3000

## 🛠️ Challenges Encountered & Solutions

## 	Challenge	                                                               ✅ Fix

1	Self-hosted runner stuck → "Listening for Jobs" but blocked terminal	  ✅ Ran sudo ./svc.sh start to launch as system service, freeing terminal

2	Git push errors due to wrong folder / credentials	✅ Navigated to correct folder nodejs-ci-cd-app before git push
✅ Used GitHub PAT instead of password

3	Invalid package.json → EJSONPARSE error	✅ Rewrote JSON properly: correct braces, double quotes, no trailing commas

4	Test module errors → missing files/dependencies	✅ Fixed test file paths (../src/server.js) ✅ Removed redundant test files
✅ Installed missing dev dependencies (chai, chai-http)

5	Server not starting → index.js missing	✅ Created src/server.js with Express app and exports for tests
✅ Started via pm2 start src/server.js

6	PM2 permission denied (EACCES)	✅ Installed with sudo npm install -g pm2
✅ Alternative: npm install pm2 --save-dev + npx pm2

7	Deploy job not running in GitHub Actions	✅ Updated ci-cd.yml with proper job dependencies (needs: test) and added deploy steps
