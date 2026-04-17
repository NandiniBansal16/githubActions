#  V-Profile DevSecOps Pipeline: Cloud-Native Deployment

##  Project Overview
This project demonstrates a fully automated, cloud-native CI/CD pipeline for a Java Spring Boot web application (V-Profile). The architecture utilizes **GitHub Actions** to automate the build, containerization, and serverless deployment of the application to **AWS ECS Fargate**. 

The goal of this project is to implement modern DevSecOps practices, eliminating manual deployments and ensuring a seamless transition from code push to a live cloud environment.

---

##  Architecture & Workflow
The pipeline follows a GitOps methodology. A push to the `main` branch triggers the following automated workflow:

1. **Source Code Checkout:** GitHub Actions pulls the latest Java source code.
2. **AWS Authentication:** Securely authenticates with AWS using GitHub Secrets (IAM Roles/Keys).
3. **Multi-Stage Containerization:** Uses a multi-stage `Dockerfile` to compile the Java code using Maven and package it into a lightweight Tomcat container.
4. **Image Registry:** Pushes the compiled Docker image to **Amazon ECR** (Elastic Container Registry).
5. **Dynamic Provisioning:** Downloads the active Task Definition directly from AWS, injecting the newly built Docker Image ID.
6. **Serverless Deployment:** Deploys the updated container to an **Amazon ECS Cluster** running on **AWS Fargate**, ensuring zero server maintenance.

---

##  Technologies & Tools Used
* **Version Control & CI/CD:** Git, GitHub Actions
* **Application Stack:** Java 17, Spring Boot, Maven, Tomcat
* **Containerization:** Docker (Multi-stage builds)
* **Cloud Infrastructure (AWS):**
  * **IAM** (Identity & Access Management for secure CI/CD roles)
  * **ECR** (Elastic Container Registry for image storage)
  * **ECS Fargate** (Serverless container orchestration)

---

##  Pipeline Details (`actions.yml`)

The CI/CD pipeline is designed as a single, highly efficient job to ensure context retention and speed:

* **No-File Task Definition:** Instead of hardcoding the `task-definition.json` in the repository, the pipeline uses the AWS CLI to dynamically fetch the live blueprint from ECS. This prevents configuration drift.
* **Service Stability Check:** The deployment step includes a `wait-for-service-stability` flag, meaning the pipeline will only show "Green/Passed" when AWS confirms the new container is healthy and serving traffic.

---

##  How to Run & Deploy

### Prerequisites
To replicate this deployment, you must have an AWS Account with the following configured:
1. An **IAM User** with permissions for ECR and ECS.
2. An **Amazon ECR Repository** named `githubactionsrepo`.
3. An **Amazon ECS Cluster** named `gitActions-server`.
4. An **ECS Task Definition** and **Service** (`vprofile-service`) running on Fargate.

### GitHub Secrets
The following secrets must be added to your GitHub Repository (`Settings > Environments > secret_env`):
* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`

### Deployment
Simply push code to the `main` branch:
```bash
git add .
git commit -m "Trigger deployment pipeline"
git push origin main
