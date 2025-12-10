Jenkins installation

Setup workflow

Agents

Pipelines

RBAC (access control)

All important commands with explanations

You can copy-paste this directly into your repo as README.md.

# 🧩 Jenkins Complete Guide – Install, Setup, Agents, Pipelines, RBAC & Commands

This document is a **one-stop Jenkins guide** for:

- ✅ Installation (Ubuntu & Docker)
- ✅ Initial setup workflow
- ✅ Agents (build nodes)
- ✅ Pipelines (Jenkinsfile)
- ✅ RBAC (Role Based Access Control)
- ✅ Important commands (with clear explanations)

---

## 📚 Table of Contents

1. [What is Jenkins?](#-what-is-jenkins)
2. [Jenkins Architecture (Controller & Agents)](#-jenkins-architecture-controller--agents)
3. [Installation](#-installation)
   - [Ubuntu (APT)](#1-ubuntu-apt-installation)
   - [Docker](#2-docker-installation)
4. [Initial Setup Workflow](#-initial-setup-workflow)
5. [Service Management & Logs](#-service-management--logs)
6. [Firewall & Port Configuration](#-firewall--port-configuration)
7. [Configuring Agents (Build Nodes)](#-configuring-agents-build-nodes)
8. [Pipelines & Jenkinsfile](#-pipelines--jenkinsfile)
9. [RBAC – Role Based Access Control](#-rbac--role-based-access-control)
10. [Jenkins CLI Commands](#-jenkins-cli-commands)
11. [Common Troubleshooting Tips](#-common-troubleshooting-tips)

---

## 📌 What is Jenkins?

Jenkins is an open-source **automation server** used for:

- Continuous Integration (CI)
- Continuous Delivery / Deployment (CD)
- Running pipelines for build, test, scan, deploy

Everything is driven via **jobs** and **pipelines**, often scripted using a `Jenkinsfile`.

---

## 🏗 Jenkins Architecture (Controller & Agents)

- **Controller (Master)**  
  - Web UI, job configuration, scheduling builds.
- **Agent (Node)**  
  - Machines that actually run the builds.
  - Can be physical, VM, Docker container, Kubernetes pod, etc.

Jenkins controller sends work to agents via SSH, JNLP, or other mechanisms.

---

## 💿 Installation

### 1️⃣ Ubuntu (APT Installation)

> These commands are for Debian/Ubuntu systems.

#### Update Packages

```bash
sudo apt-get update


🔹 Updates the list of available packages & versions.

Install Java (Required by Jenkins)
sudo apt-get install -y openjdk-17-jdk


🔹 Installs Java 17, which Jenkins uses to run.

Add Jenkins Repository Key
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null


🔹 Adds Jenkins signing key so APT trusts Jenkins packages.

Add Jenkins Repository
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null


🔹 Tells APT where to fetch Jenkins from.

Install Jenkins
sudo apt-get update
sudo apt-get install -y jenkins


🔹 Downloads and installs Jenkins service.

2️⃣ Docker Installation

Good for local/dev environments or containerized CI.

Pull Jenkins LTS Image
docker pull jenkins/jenkins:lts


🔹 Downloads the latest stable Jenkins image.

Run Jenkins Container
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts


Explanation:

-d → Runs container in background (detached).

-p 8080:8080 → Exposes Jenkins web UI on port 8080.

-p 50000:50000 → Used for JNLP agents.

-v jenkins_home:/var/jenkins_home → Persists Jenkins data.

🚀 Initial Setup Workflow

After installation:

1️⃣ Access Jenkins UI

Open browser and go to:

http://<server-ip>:8080


Example: http://localhost:8080 or http://your-server-ip:8080

2️⃣ Get Initial Admin Password

UBUNTU:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword


DOCKER:

docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword


🔹 Use this in the web UI to unlock Jenkins.

3️⃣ Install Suggested Plugins

In UI, after unlock → choose “Install suggested plugins”.

Jenkins will install basics like Git, Pipeline, etc.

4️⃣ Create First Admin User

Set a username, password, full name, and email.

This will be your main login for Jenkins.

5️⃣ Configure Jenkins URL

In UI:

Manage Jenkins → System → Jenkins URL

Set it to the actual URL you use, e.g.:

http://your-ip:8080/

or https://your-domain.com/jenkins/

🔹 This fixes issues like wrong redirects & reverse proxy warnings.

🔁 Service Management & Logs
Start Jenkins
sudo systemctl start jenkins


🔹 Starts Jenkins service.

Enable Jenkins on Boot
sudo systemctl enable jenkins


🔹 Automatically starts Jenkins at system boot.

Stop Jenkins
sudo systemctl stop jenkins


🔹 Stops the service.

Restart Jenkins
sudo systemctl restart jenkins


🔹 Useful after plugin installs & configuration changes.

Check Jenkins Status
sudo systemctl status jenkins


🔹 Shows if Jenkins is active, inactive, or failed.

Logs (Debugging)
View Live Logs
sudo journalctl -u jenkins -f


🔹 -f = follow logs in real time.

View Log File Directly
cat /var/log/jenkins/jenkins.log


🔹 Shows complete Jenkins log output.

🔐 Firewall & Port Configuration

If you use UFW firewall:

Allow Jenkins (Port 8080)
sudo ufw allow 8080


🔹 Opens port 8080 for external access.

Check Firewall Rules
sudo ufw status


🔹 Shows allowed/denied ports.

Check Which Process is Using Port 8080
sudo ss -tulnp | grep 8080


Columns:

Port → 8080

Process name → e.g. jenkins

PID → process ID

🌐 Configuring Agents (Build Nodes)

Agents are extra machines that run builds to offload work from the controller.

1️⃣ Create a New Node (Agent) in UI

Go to Manage Jenkins → Nodes & Clouds → New Node

Give it a name (e.g. agent-1)

Select Permanent Agent

Configure:

# of executors → how many parallel builds

Remote root directory → e.g. /home/jenkins

Labels → e.g. docker, java, linux

Launch method → usually “Launch agents via SSH”

2️⃣ SSH Agent Setup (Common Case)

On the agent machine:

Create Jenkins User
sudo useradd -m -s /bin/bash jenkins


🔹 Creates user jenkins with home directory.

Allow SSH Login
sudo passwd jenkins


🔹 Set password for Jenkins user (if using password auth).

Or better: use SSH keys.

3️⃣ Controller → Agent SSH

On controller, generate key (if not existing):

ssh-keygen -t rsa -b 4096


🔹 Generates a new SSH key.

Copy public key to agent:

ssh-copy-id jenkins@<agent-ip>


🔹 Allows password-less SSH from controller to agent.

Then in Jenkins UI → Node config:

Host: <agent-ip>

Credentials: SSH username + private key

4️⃣ Docker Agent (Simple Example)

If you want an agent inside Docker:

docker run -d \
  --name jenkins-agent \
  -e JENKINS_URL=http://<controller-ip>:8080 \
  -e JENKINS_AGENT_NAME=agent-docker \
  -e JENKINS_SECRET=<secret-from-node-config> \
  jenkins/inbound-agent


🔹 Used when using inbound (JNLP) agents.

🧪 Pipelines & Jenkinsfile

Pipelines are defined in a file named Jenkinsfile stored in your repo.

Basic Declarative Pipeline Example
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker run -d -p 8080:8080 --name myapp myapp:latest'
            }
        }
    }
}

Key Keywords:

pipeline {} → Top level pipeline block.

agent any → Run on any available node.

stages {} → Contains all stages of the CI/CD.

stage('Name') {} → A logical step in pipeline (build, test, deploy).

steps {} → Shell commands or Jenkins steps.

sh 'command' → Runs a shell command on the agent.

👥 RBAC – Role-Based Access Control

You can control who can see/do what in Jenkins.

1️⃣ Enable Security Realm (Users)

In UI:

Go to Manage Jenkins → Security → Configure Global Security

Set Security Realm:

“Jenkins’ own user database”

Check Allow users to sign up (optional)

Save and create users if needed.

2️⃣ Authorization Strategy

Two common options:

Matrix-based security (built-in)

Role-Based Strategy (via plugin)

A) Matrix-based security (Built-in)

Go to Manage Jenkins → Configure Global Security

Under Authorization → select Matrix-based security

Add users/groups

Grant permissions such as:

Overall → Read, Administer

Job → Read, Build, Configure

View → Read

Credentials → Create, Update, View

Click Save

B) Role-Based Strategy (Plugin)

You must install Role-based Authorization Strategy plugin.

Steps:

UI → Manage Jenkins → Manage Plugins → Available

Search for: Role-based Authorization Strategy

Install & restart Jenkins if necessary.

Then:

Manage Jenkins → Configure Global Security

Under Authorization → choose Role-Based Strategy

Save

Next, define roles:

Go to Manage Jenkins → Manage and Assign Roles → Manage Roles

Create roles:

admin – full access

dev – create and run jobs, but not admin

viewer – read-only access

Assign permissions per role (Overall, Job, View etc.)

Assign roles to users:

Go to Manage Jenkins → Manage and Assign Roles → Assign Roles

Map users to roles (global + project roles).

🧮 Jenkins CLI Commands

Download CLI jar:

wget http://<jenkins-url>/jnlpJars/jenkins-cli.jar


Login using API token (recommended).

Reload Configuration
java -jar jenkins-cli.jar -s http://<jenkins-url> reload-configuration


🔹 Reloads Jenkins configuration from disk without full restart.

Safe Restart
java -jar jenkins-cli.jar -s http://<jenkins-url> safe-restart


🔹 Allows current builds to finish, then restarts Jenkins.

List Jobs
java -jar jenkins-cli.jar -s http://<jenkins-url> list-jobs


🔹 Lists all Jenkins jobs.

Build a Job
java -jar jenkins-cli.jar -s http://<jenkins-url> build <job-name>


🔹 Triggers a build for the given job.

🧯 Common Troubleshooting Tips
1️⃣ Reverse Proxy Warning:

“It appears that your reverse proxy setup is broken”

✅ Fix by ensuring:

Jenkins URL matches browser URL

Reverse proxy (Nginx/Apache) sends correct headers:

Host

X-Forwarded-Proto

X-Forwarded-For

2️⃣ Port 8080 Already in Use

Check process:

sudo ss -tulnp | grep 8080


Kill process (careful):

sudo kill -9 <PID>


Restart Jenkins:

sudo systemctl restart jenkins

3️⃣ Backup Jenkins
sudo cp -r /var/lib/jenkins /var/lib/jenkins_backup


🔹 Backs up jobs, plugins, config.

Restore:

sudo rm -rf /var/lib/jenkins
sudo cp -r /var/lib/jenkins_backup /var/lib/jenkins

✅ Summary

This README covered:

Installation (Ubuntu & Docker)

Setup workflow (unlock, plugins, URL)

Service management & logs

Agents (SSH & Docker)

Pipelines with Jenkinsfile

RBAC using Matrix & Role-Based Strategy

CLI & useful admin commands

Troubleshooting patterns

You can extend this repo with:

Example Jenkinsfile

Nginx reverse proxy config

DevSecOps pipeline (Trivy, SonarQube, ZAP, Docker Scout)
