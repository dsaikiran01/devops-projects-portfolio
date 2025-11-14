# Project-2: CI/CD Pipeline using Jenkins

![Source: GitHub](https://img.shields.io/badge/Source-GitHub-blue?logo=github)
![Cloud: AWS](https://img.shields.io/badge/Cloud-AWS-orange?logo=amazon-aws)
![Build Tool: Maven](https://img.shields.io/badge/Build_Tool-Maven-yellow?logo=apache-maven)
![Application Server: Tomcat](https://img.shields.io/badge/App_Server-Tomcat-green?logo=apache-tomcat)
![CI/CD: Jenkins](https://img.shields.io/badge/CI%2FCD-Jenkins-red?logo=jenkins)


## Project Title
**CI/CD Pipeline for Java Web Application Using Jenkins, Maven & Tomcat**


* In this project, we implement a complete **CI/CD pipeline** for deploying a Java-based web application using **GitHub, Jenkins, Maven and Tomcat**.  
* Whenever new code is pushed to GitHub, Jenkins automatically builds the application and deploys the generated WAR file to a remote Tomcat server.


## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Technologies Used](#technologies-used)
- [Project Workflow](#project-workflow)
- [Step-by-Step Implementation](#step-by-step-implementation)
  - [1. Prepare GitHub Repository](#1-prepare-github-repository)
  - [2. Create Jenkins Server](#2-create-jenkins-server)
  - [3. Install Java and Jenkins](#3-install-java-and-jenkins)
  - [4. Configure Jenkins](#4-configure-jenkins)
  - [5. Create Tomcat Server](#5-create-tomcat-server)
  - [6. Install & Configure Tomcat](#6-install--configure-tomcat)
  - [7. Configure Jenkins Deployment](#7-configure-jenkins-deployment)
  - [8. Create Jenkins Freestyle Job](#8-create-jenkins-freestyle-job)
  - [9. Trigger the Pipeline](#9-trigger-the-pipeline)
  - [10. Setup GitHub Webhook](#10-setup-github-webhook)
  - [11. Testing the CI/CD Pipeline](#11-testing-the-cicd-pipeline)
  - [12. Cleanup the Setup](#12-cleanup-the-setup-remove-project-resources)
- [How CI/CD Works](#how-cicd-works)
- [Conclusion](#conclusion)


## Overview

This project demonstrates how to build a complete **Continuous Integration and Continuous Deployment (CI/CD)** setup using:

- **GitHub**: Source code repository  
- **Jenkins**: CI/CD automation server  
- **Maven**: Build automation tool  
- **Apache Tomcat**: Application deployment server  

The pipeline automatically builds a WAR file and deploys it to a Tomcat server whenever a push is made to the GitHub repository.


## Architecture

![Project Architecture](./assets/01-Architecture.png)

**Workflow Summary**  
Developer → Push Code to GitHub → GitHub Webhook → Jenkins → Maven Build → WAR File → Remote Tomcat → Application Live


## Technologies Used

- **Jenkins**
- **GitHub**
- **Maven**
- **Apache Tomcat 9**
- **Java (OpenJDK)**
- **Ubuntu EC2 Servers**


## Project Workflow

1. Developer commits code to GitHub  
2. GitHub Webhook triggers Jenkins  
3. Jenkins pulls new code  
4. Jenkins builds the application using Maven  
5. Jenkins deploys the WAR file to Tomcat using “Deploy to Container” plugin  
6. Tomcat automatically deploys the application  
7. Application becomes live at `http://TOMCAT-IP:8090/app-name`


# Step-by-Step Implementation

## 1. Prepare GitHub Repository
- Add your Java web application source code.
- Ensure the project contains a `pom.xml` file for Maven.

![GitHub Repo](./assets/02-Github-repo.png)


## 2. Create Jenkins Server

Launch an EC2 instance for Jenkins:

- **Name**: `Jenkins-server`  
- **AMI**: Ubuntu  
- **Instance Type**: `c7i-flex.large` (Jenkins requires high CPU/RAM)  
- **Security Group**: Allow **all inbound** temporarily for project use  
- **Key Pair**: Your key  

![Jenkins Server Ready](./assets/03-jenkins-server.png)


## 3. Install Java and Jenkins

SSH into the Jenkins server and run install java:
```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre -y
java -version # check installation
````

Install Jenkins:
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
```

Get latest commands from here: `https://www.jenkins.io/doc/book/installing/linux/#debianubuntu`

Start Jenkins:

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins # check if running
```

![Jenkins runs](./assets/04-check-jenkins-install.png)


## 4. Configure Jenkins (UI)

Access Jenkins at `http://JENKINS-IP:8080`

Steps:

1. Unlock Jenkins using initial admin password
2. Install **Suggested Plugins**
3. Create **Admin User**
4. Install **Maven**
   * Navigate: `Manage Jenkins → Tools → Maven → Add Maven`

![Step 3](./assets/05-create-jenkins-user.png)

![Jenkins Ready](./assets/06-jenkins-ready.png)

![Jenkins Homepage](./assets/07-Jenkins-UI.png)

![Navigate to tools](./assets/08-jenkins-tools.png)

![Add maven and details](./assets/09-maven-install.png)


## 5. Create Tomcat Server

Launch another EC2 instance:

* **Name**: `tomcat-server`
* **AMI**: Ubuntu
* **Instance Type**: `t3.micro`
* **Security Group**: Allow all inbound (project mode)

![Tomcat Server Ready](./assets/10-tomcat-server.png)


## 6. Install & Configure Tomcat

SSH into the Tomcat server.

Install Java:

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
java -version
```

![Java installed](./assets/11-install-java.png)

Download Tomcat:

1. Go to [Apache](https://tomcat.apache.org/download-90.cgi) website
2. Copy the `.tar.gz` download link

![Tomcat download links](./assets/12-copy-tomcat-link.png)

3. Run to download and extract tarball:

```bash
wget <TOMCAT-DOWNLOAD-LINK>
tar -xvzf apache-tomcat*.tar.gz
```

![Downloaded tomcat](./assets/13-download-tomcat.png)

![Extract tarball](./assets/14-extract-tomcat-tar.png)

Start Tomcat:

```bash
cd apache-tomcat*/bin
./shutdown.sh # recommended prior to starting
./startup.sh
```

Access:
```
http://TOMCAT-IP:8080
```

![Tomcat web UI](./assets/15-access-tomcat.png)

### Fix Port Conflict (Jenkins also uses 8080)

1. Edit `server.xml`:
```bash
sudo nano conf/server.xml
```

2. Change connector port:
```xml
<Connector port="8090" ... />
```

3. Restart Tomcat:
```bash
./shutdown.sh
./startup.sh
```

![Tomcat port changed](./assets/16-tomcat-port-changed.png)

### Configure Tomcat Users

1. Edit `tomcat-users.xml`:
```bash
sudo nano conf/tomcat-users.xml
```

2. Add:
```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
<user username="deployer" password="deployer" roles="manager-script"/>
```


### Grant External Access to Manager App

1. Find `context.xml`:
```bash
sudo find / -name context.xml
```

![Manager's context file](./assets/17-edit-context-file.png)

2. Comment out the `<Valve>` tag.

![Comment tag](./assets/18-comment-valve.png)

3. Restart Tomcat and sign-in to manager console.

![Sign in with admin credentials](./assets/19-sign-in-manager.png)

![Manager Console UI](./assets/20-Access-manager-console.png)


## 7. Configure Jenkins Deployment

In Jenkins:

1. Install **Deploy to Container** plugin

![Plugin Install](./assets/21-install-plugin.png)

2. Restart Jenkins
3. Go to: `Manage Jenkins → Credentials → Global`
4. Add credentials for:

   * **Username**: `deployer`
   * **Password**: `deployer`

![All global credentials](./assets/22-go-to-global-creds.png)

![Add new credential](./assets/23-add-global-cred.png)

![Create credential](./assets/24-create-global-cred.png)


## 8. Create Jenkins Freestyle Job

Create Job: **Shopping-site-deploy**

![New Project](./assets/25-create-new-job.png)

Configure:

### Source Code Management

* GitHub Repo URL
* Branch: `main` or `master`

![Configure source code](./assets/26-give-git-repo.png)

### Build Trigger

* Enable **GitHub hook trigger for GITScm polling**

![Trigger setup](./assets/27-select-trigger.png)

### Build Step

Add:

* “Invoke top-level Maven targets”
* Maven Version: selected from settings
* Goals:
  ```
  clean install package
  ```

![Build step](./assets/28-add-build-step.png)

![Add Maven](./assets/29-configure-maven.png)

### Post-build Action

Select **Deploy war/ear to a container**

* WAR file path:
  ```
  **/*.war
  ```
* Add container:

  * Credentials: `deployer`
  * Tomcat URL:
    ```
    http://TOMCAT-IP:8090/
    ```

Apply → Save

![Post Build Action](./assets/30-add-post-build-action.png)

![Configure post build](./assets/31-configure-post-build-action.png)


## 9. Trigger the Pipeline

You need to manually trigger initial build:

* Jenkins → Job → **Build Now**

After that:

* Every push to GitHub triggers the Jenkins job automatically
* The WAR file is deployed into Tomcat’s `webapps` folder

![Click Build Now](./assets/32-Test-pipeline.png)

![Deployed site files](./assets/33-war-added.png)

![Live page](./assets/34-site-deployed.png)


## 10. Setup GitHub Webhook

Go to GitHub Repository → **Settings → Webhooks → Add Webhook**

![Repo's webhook page](./assets/35-add-webhook.png)

Add:
  * **Payload URL**:
    ```
    http://JENKINS-IP:8080/github-webhook/
    ```
  * **Content type**: application/json
  * **Trigger**: “Just the push event”
  * Save

![Webhook configuration](./assets/36-configure-webhook.png)

![Webhook created](./assets/37-webhook.png)


## 11. Testing the CI/CD Pipeline

* After completing the setup, it’s time to verify that the CI/CD pipeline works end-to-end.  
* This test ensures that whenever code changes are pushed to GitHub, Jenkins automatically builds and deploys the updated application to the Tomcat server.

1. Open your project locally on your system.
2. Modify any file in the application—for example, update a line in `index.jsp`.
3. Save the file.
4. Commit the change:
5. Push the changes to GitHub

* Final Application URL
```
http://TOMCAT-SERVER-IP:8090/shopping-site-web-app
```

![Code changes and commit](./assets/38-code-changes-commit.png)

![Jenkins triggered by webhook](./assets/39-jenkins-trigger.png)

![Live page with changes](./assets/40-changes-deployed.png)


## 12. Cleanup the Setup: Remove Project Resources

Once testing is complete and you’ve confirmed that the CI/CD pipeline works as expected, you can safely delete the temporary resources used for this project to avoid unnecessary AWS costs.

1. Delete GitHub Webhook
2. Terminate EC2 Instances
3. Delete Attached EBS Volumes (If Not Automatically Removed)

![step-12-1](./assets/41-delete-webhook.png)

![step-12-2](./assets/42-delete-instances.png)


# How CI/CD Works

1. Developer commits code
2. GitHub triggers Jenkins via webhook
3. Jenkins builds using Maven
4. WAR file generated
5. Jenkins deploys WAR to Tomcat
6. Tomcat host auto-extracts it
7. Updated app becomes live instantly


# Conclusion

In this project, we build a complete **Jenkins-based CI/CD pipeline**, packaging and deploying a Java web application to Apache Tomcat. The entire process is automated using GitHub → Jenkins → Tomcat, making deployments reliable, repeatable, and fast.
