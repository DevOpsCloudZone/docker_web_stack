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

### Amazon Linux 2023

```bash
sudo yum install fontconfig java-21-amazon-corretto -y
```
### Red Hat / CentOS

```bash
sudo yum install fontconfig java-21-openjdk -y
```
### Ubuntu / Debian

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
