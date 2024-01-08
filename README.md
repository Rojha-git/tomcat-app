## Build and Deploy a Modern YouTube Clone Application in React JS with Material UI 5

 ![Complete CI-CD]()


## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Step-by-Step Guide](#step-by-step-guide)
  - [1. Create an EC2 Instance on AWS](#1-create-an-ec2-instance-on-aws)
  - [2. Install Terraform](#2-install-terraform)
  - [3. Clone the Terraform Scripts](#3-clone-the-terraform-scripts)
  - [4. Check the Status of Installed Services](#4-check-the-status-of-installed-services)
  - [5. Login to Jenkins Console](#5-login-to-jenkins-console)
  - [6. Configure Jenkins and Install Plugins](#6-configure-jenkins-and-install-plugins)
  - [7. Generate Access Tokens](#7-generate-access-tokens)
  - [8. Create Jenkins Pipeline](#8-create-jenkins-pipeline)
  - [9. SonarQube Configuration](#9-sonarqube-configuration)
  - [10. Setup Monitoring with Prometheus and Grafana](#10-setup-monitoring-with-prometheus-and-grafana)
  - [11. Jenkins Pipeline Execution](#11-jenkins-pipeline-execution)
  - [12. Create AWS EKS Cluster](#12-create-aws-eks-cluster)
  - [13. Integrate Prometheus with EKS and Import Grafana Dashboard](#13-integrate-prometheus-with-eks-and-import-grafana-dashboard)
  - [14. Automated Deployment with Jenkins on Kubernetes](#14-automated-deployment-with-jenkins-on-kubernetes)
  - [15. Webhook Configuration](#15-webhook-configuration)
- [Conclusion](#conclusion)
- [Acknowledgements](#acknowledgements)
- [License](#license)

## Introduction
This repository provides a step-by-step guide for building and deploying a modern YouTube clone application using React JS with Material UI 5. The guide covers setting up infrastructure on AWS, configuring Jenkins for CI/CD, integrating SonarQube for code analysis, and implementing monitoring with Prometheus and Grafana.

## Prerequisites
Before you begin, make sure you have the following prerequisites:
- AWS account
- GitHub account
- DockerHub account
- SonarQube account
- Gmail account for email notifications

## Step-by-Step Guide

1.create an ec2(t2.micro) instance on AWS and after ssh to this instance install terraform on that using below commands:
    #Confirm the latest version number on the terraform website:
    https://www.terraform.io/downloads.html
    
      wget https://releases.hashicorp.com/terraform/1.0.7/terraform_1.0.7_linux_amd64.zip

      apt install unzip
    
      unzip terraform_1.0.7_linux_amd64.zip
    
      sudo mv terraform /usr/local/bin/
    
      terraform --version


2.clone the git repo for terraform scripts-


      git clone "https://github.com/Rojha-git/Terraform_YT_cloned_app.git"
   
      cd Terraform_YT_cloned_app        #under this we have two repo so choose one by one and run below command.
   
      terraform init                    # aws should be configure with your instances
   
      terraform plan
   
      terraform apply
   
    
3.check the status of the installed services:


   check jenkins [http://<ip_addr_jenkins_server>:8080](http://<ip_addr_jenkins_server>:8080)

   check sonar: http://<ip_addr_jenkins_server>:9000
   
   check prometheus: http://<ip_addr_monitoring_server>:9090  #if url is not came up check status of service if not running start it using

   ```bash

   $ sudo systemctl start prometheus

   ```
   check grafana []http://<ip_addr_monitoring_server>:3000

   --Commands to check services

   ```bash
   
   $ sudo systemctl status prometheus
   
   $ sudo systemctl status node_exporter
   
   $ sudo systemctl status grafana-server

   ```

4.login to jenkins console:
   
   using http://<ip_addr_jenkins_server>:8080
   
   primary password you can copy from the server using below commad :
   ```bash

   sudo cat /var/lib/jenkins/secrets/initialAdminPassword

   ``` 
   - Enter the Administrator password
   
   after that you can create your userid and password.

5. after login to jenkins first of all click on "manage plugins" and install required plugins for **"Docker, Sonar, maven , eclipse, quality gates , prometheus"** and restart jenkins.
   also under the "tools" section configure jdk , maven , sonar scanner and
   under "system>>sonarqube-installation>> "configure sonar server using http://<privateip_sonar_server>:9000 console urls.
   
   under "manage jenkins>>tools>>sonarqube installation>>" configure sonar tool with latest version . 

7. Generate the access token from sonarqube(secret_text) , github , dockerhub and configure all this under the jenkins "credentials" so that jenkins can access all these.

8. Under jenkins create an pipeline with "tomcat-app-ci-cd" name using below steps:

     --- select SCM option and provide repo [https://github.com/Rojha-git/tomcat-app.git](https://github.com/Rojha-git/tomcat-app.git) to access jenkins file for ci-cd.

9. login to sonarqube console and configure quality gate , webhook , once all these completed then after the jenkins pipeline 
    completion you will be able to see all the flow and unit tests.

10. Login to prometheus using http://<ip_addr_monitoring_server>:9090   #username and password will be "admin"
    after login :
    
    --Add job for node exporter in prometheus
   
    ```bash
    
    $ cd /etc/prometheus/
   
    $ sudo nano prometheus.yml

    ```
    #add job for node exporter abd jenkins
         
    
         - job_name: 'node_exporter'
           static_configs:
             - targets: ['IP-Address-monitoring:9100']

    
         - job_name: 'jenkins'
           metrics_path: '/prometheus'
           static_configs:
             - targets: ['IP-Address-jenkins:8080']
    
         
   
    ```bash
    
    #Check the indentatio of the prometheus config file with below command
    
    $ promtool check config /etc/prometheus/prometheus.yml

    #Reload the Prometheus configuration
    
    $ curl -X POST http://localhost:9090/-/reload

    ```
11. After performing the #10 point you will be able to see the targets for matrices under the traget option in prometheus console.

12. login to grafana using http://<ip_addr_monitoring_server>:3000 --> #username and password will be "admin"

    --configure prometheus as the data source under the grafana using the prometheus url:

    --Import or add the dashboard for prometheus and jenkins by dashboard id of grafana   # you can try by google as well for dashboard id.


    **** Now you are able to access the monitoring console on grafana for both jenkins job and node_exporter server  ****
