⭐ Jenkins README – Installation, Management & Common Commands (With Explanations)

Jenkins is an open-source automation server used for CI/CD pipelines.
This README explains every important Jenkins command in a clear and simple way so that when you see it later, you instantly know what it is for.

📌 1. Install Jenkins on Ubuntu
Update package index
sudo apt-get update


✔ Updates the list of packages so your system knows the latest versions.

Install Java (Jenkins requires Java)
sudo apt install openjdk-17-jdk -y


✔ Installs Java 17, required for Jenkins to run.

Add Jenkins repository key
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null


✔ Adds Jenkins official signing key so your system trusts Jenkins packages.

Add Jenkins repository to sources list
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null


✔ Tells Ubuntu where to download Jenkins from.

Install Jenkins
sudo apt-get update
sudo apt-get install jenkins -y


✔ Downloads and installs Jenkins server.

Start Jenkins service
sudo systemctl start jenkins


✔ Starts the Jenkins service.

Enable Jenkins service at boot
sudo systemctl enable jenkins


✔ Makes Jenkins start automatically when the system boots.

📌 2. Check Jenkins Status
Check if Jenkins is running
sudo systemctl status jenkins


✔ Shows whether Jenkins is active, running, or stopped.

Restart Jenkins service
sudo systemctl restart jenkins


✔ Useful after installing plugins or changing configuration.

Stop Jenkins
sudo systemctl stop jenkins


✔ Stops Jenkins service.

📌 3. Jenkins Logs Commands
View Jenkins logs
sudo journalctl -u jenkins -f


✔ Shows live Jenkins logs (helpful for debugging errors).

View full Jenkins log file
cat /var/log/jenkins/jenkins.log


✔ Shows all past logs stored in Jenkins log file.

📌 4. Jenkins Default Credentials Setup
Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword


✔ Displays the password required to unlock Jenkins after installation.

📌 5. Firewall Commands (UFW)

If UFW firewall is enabled, Jenkins (port 8080) must be allowed.

Allow Jenkins Port (8080)
sudo ufw allow 8080


✔ Allows external access to Jenkins UI.

Check Firewall Status
sudo ufw status


✔ Shows which ports are allowed or blocked.

📌 6. Reverse Proxy Commands (Nginx)

If using Nginx in front of Jenkins:

Install Nginx
sudo apt-get install nginx -y


✔ Installs Nginx web server.

Edit Nginx config
sudo nano /etc/nginx/sites-available/jenkins


✔ Creates a separate config file for Jenkins.

Check Nginx config syntax
sudo nginx -t


✔ Ensures no errors before restarting.

Restart Nginx
sudo systemctl restart nginx


✔ Applies the new configuration.

📌 7. Jenkins CLI Useful Commands
Reload Jenkins configuration
java -jar jenkins-cli.jar -s http://localhost:8080 reload-configuration


✔ Reloads Jenkins without restarting service.

Safe restart Jenkins using CLI
java -jar jenkins-cli.jar -s http://localhost:8080 safe-restart


✔ Gracefully restarts Jenkins without killing builds.

📌 8. Backup & Restore Jenkins
Backup Jenkins Home Folder
sudo cp -r /var/lib/jenkins /var/lib/jenkins_backup


✔ Saves all jobs, plugins, and configurations.

Restore Jenkins Backup
sudo rm -rf /var/lib/jenkins
sudo cp -r /var/lib/jenkins_backup /var/lib/jenkins


✔ Restores Jenkins to previous state.

📌 9. Upgrade Jenkins
Upgrade Jenkins using APT
sudo apt-get update
sudo apt-get install jenkins -y


✔ Installs the latest version while keeping your data safe.

📌 10. Docker Commands (If Running Jenkins in Docker)
Pull Jenkins LTS image
docker pull jenkins/jenkins:lts


✔ Downloads latest Jenkins stable image.

Run Jenkins in Docker
docker run -d -p 8080:8080 -v jenkins_home:/var/jenkins_home --name jenkins jenkins/jenkins:lts


✔ Starts Jenkins container with persistent data.

Check running containers
docker ps


✔ Shows Jenkins container status.

Stop Jenkins container
docker stop jenkins

Start Jenkins container
docker start jenkins

📌 11. Check Ports Used by Jenkins
Find which process is using port 8080
sudo ss -tulnp | grep 8080


✔ Useful when Jenkins fails to start due to port being used.

📌 12. Jenkins URL Fix (Reverse Proxy Error)
Where to fix Jenkins URL

Manage Jenkins → Configure System → Jenkins URL

✔ Must match the domain or IP you use in the browser (fixes reverse proxy error).

🎉 README Summary

This README covers:

✔ Jenkins installation
✔ Service commands
✔ Logging & debugging
✔ Firewall config
✔ Reverse proxy
✔ Backup & restore
✔ Docker version
✔ Port checks
✔ Upgrade procedure

If you want, I can also create:

📌 README for Jenkins Pipelines (Declarative + Scripted)
📌 README for DevSecOps tools (Trivy, SonarQube, ZAP, Docker Scout)
📌 README for Jenkins + Docker + GitHub integration

Just tell me: "Create pipeline README" or specify which one.

You said:
yes
ChatGPT said:

Great! Since you said “yes”, I will create a complete Jenkins Pipeline README for you.
This README will explain:

What is a Jenkins pipeline

Stages and how they work

Declarative vs Scripted pipeline

Example Jenkinsfiles

Explanation of every command clearly

Perfect for GitHub, resume, and learning.

⭐ Jenkins Pipeline README (With Explanation of Every Command)

A Jenkins Pipeline is a set of automated steps that run inside Jenkins to build, test, scan, and deploy your application.
This README teaches you everything about Pipelines in a simple way.

📌 1. What Is a Jenkins Pipeline?

A Jenkins Pipeline is:

A script written in Jenkinsfile

Used to automate build → test → scan → deploy

Stored in your project so CI/CD becomes repeatable

A pipeline contains Stages and Steps:

Stage = Big block of work (Build, Test, Deploy)

Step = A single command inside a stage

📌 2. Pipeline Types
A) Declarative Pipeline (Recommended)

Easy, structured, modern.

B) Scripted Pipeline

Advanced, flexible; uses Groovy scripting.

This README uses Declarative Pipeline.

⭐ 3. Basic Declarative Pipeline (With Command Explanation)
pipeline {
    agent any


✔ agent any → Run pipeline on any Jenkins node.

📌 Checkout Stage
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }


✔ checkout scm → Pulls code from GitHub/GitLab linked to the job.
✔ This ensures Jenkins always builds the latest code.

📌 Build Stage
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }


✔ sh → Used to run shell commands.
✔ mvn clean install → Cleans old files and builds the application.

📌 Test Stage
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }


✔ Runs all automated test cases.

📌 Package Stage (For Docker)
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }


✔ Builds a Docker image using your Dockerfile.
✔ -t myapp:latest → Tags the image.

📌 Push Image to Docker Hub
        stage('Push to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-pass', variable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login -u mydockerid --password-stdin
                        docker push myapp:latest
                    '''
                }
            }
        }


✔ withCredentials → Reads secret values from Jenkins.
✔ docker login → Logs into Docker registry.
✔ docker push → Uploads the image to Docker Hub.

⭐ 4. Full Working Jenkinsfile (Copy-Paste Ready)
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
                sh 'mvn clean install'
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

        stage('Push to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'docker-pass', variable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login -u mydockerid --password-stdin
                        docker push myapp:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker run -d -p 8080:8080 myapp:latest'
            }
        }
    }
}

⭐ 5. Pipeline Commands Explained
Command	Meaning
pipeline {}	Starting block of Jenkinsfile
agent any	Run on any available node
stages {}	Group of all pipeline stages
stage('Name')	Logical block (Build/Test/Deploy)
steps {}	Commands inside a stage
sh 'command'	Execute Linux command
checkout scm	Pull code from Git repository
withCredentials	Access secret values
docker build	Build Docker image
docker push	Push image to registry
docker run	Run container
⭐ 6. Pipeline With DevSecOps Tools (Trivy + Sonar + ZAP)

If you want a DevSecOps README, here is the pipeline summary:

Tool	Stage	Purpose
SonarQube	Code scan	Code quality & security
Trivy FS	Pre-build	Scan source code for CVE & secrets
Trivy Image	After docker build	Scan docker image
Docker Scout	Image review	Vulnerability analysis
OWASP ZAP	After deployment	Attack running app

I can include complete commands if you want.

⭐ 7. How to Run a Jenkins Pipeline Job
Step 1: Create a new pipeline job

Inside Jenkins → New Item → Pipeline

Step 2: Select SCM

Choose Git and enter repository URL.

Step 3: Add Jenkinsfile

Either:

Put Jenkinsfile in your repository

OR paste into pipeline editor

Step 4: Build the job

Click Build Now.

🎉 README is ready!

This README explains:

Pipeline basics

Stages

Commands (with meaning)

Complete Jenkinsfile

Git + Docker integration

If you want, I can create:

🔹 README for DevSecOps Jenkins Pipeline
🔹 README for Docker + Jenkins
🔹 README for Jenkins Installation + Troubleshooting

Just tell me:
👉 “Create DevSecOps README” or
👉 “Create Docker Jenkins README”
