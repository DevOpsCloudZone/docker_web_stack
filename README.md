# Docker Web Stack

## Project Overview

This project implements an end-to-end CI/CD pipeline for a Java web application using Jenkins, Maven, SonarQube, Nexus, Docker, Docker Hub, and Docker Swarm.

## Project Architecture

📄 **[View Project Architecture & CI/CD Pipeline](./Project-Architecture-and-CICD-Pipeline.pdf)**

## Documentation

📘 **[Docker Push Stage – Detailed Explanation](./Docker-Push-Stage-Explanation.md)**

## Jenkins Installation on Amazon Linux 2023

### Step 1: Add the Jenkins Repository

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/rpm-stable/jenkins.repo

sudo yum upgrade -y
```

## Step 2: Install Java 21 and Required Dependencies

**Amazon Linux 2023**

```bash
sudo yum install fontconfig java-21-amazon-corretto -y
```
**Red Hat / CentOS**

```bash
sudo yum install fontconfig java-21-openjdk -y
```
**Ubuntu / Debian**

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre -y
```
Verify the Java version:

```bash
java -version
```

### Step 3: Install Jenkins

```bash
sudo yum install jenkins -y
```

Verify the Jenkins version:

```bash
jenkins --version
```

### Step 4: Start and Enable Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```

### Step 5: Access Jenkins

Allow inbound TCP traffic on port `8080` in the EC2 instance's Security Group.

Open Jenkins in your browser:

```text
http://<EC2-PUBLIC-IP>:8080
```

Retrieve the initial administrator password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Enter the password in the Jenkins setup page, install the suggested plugins, and create your administrator account.

## SonarQube Installation on EC2

### Step 1: Create a New EC2 Server

Create a new EC2 instance for SonarQube installation.

### Step 2: Install Docker

```bash
sudo yum install docker -y
```

Start and enable Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

Verify Docker installation:

```bash
docker -v
```

### Step 3: Pull SonarQube Docker Image

Pull the SonarQube image from Docker Hub:

```bash
docker pull sonarqube
```

Verify the downloaded image:

```bash
docker images
```

### Step 4: Run SonarQube Container

Run SonarQube and map port `9000`:

```bash
docker run -d --name sonarqube-cont -p 9000:9000 sonarqube:latest
```

Verify that the container is running:

```bash
docker ps
```

### Step 5: Allow Port 9000 in EC2

Make sure **TCP port 9000** is allowed in the SonarQube EC2 instance's **Security Group → Inbound Rules**.

### Step 6: Access SonarQube

Open SonarQube in your browser:

```text
http://<EC2-PUBLIC-IP>:9000
```

### Step 7: SonarQube Initial Login

Use the default credentials:

username:
```bash
admin
```
Password:
```bash
admin
```

After the first login, update the password.

### Step 8: Create SonarQube Project

From the SonarQube UI:

```text
Create a local project
```

Enter your project name and create the project.

### Add the Jenkins webhook URL:

```text
http://<JENKINS-PUBLIC-IP>:8080/sonarqube-webhook/
```

This webhook allows Jenkins to receive the SonarQube Quality Gate result.

### Step 9 : Verify SonarQube

After the Jenkins pipeline runs, verify the code-quality analysis results in the SonarQube UI.




