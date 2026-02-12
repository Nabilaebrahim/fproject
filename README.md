# 🚀 3-Tier Web Application Deployment (DevOps Project)

## 📝 Project Overview
This project demonstrates a professional **End-to-End DevOps Pipeline** for a 3-tier application. It focuses on automating the deployment process using modern Cloud-Native tools, ensuring high availability, security policies, and real-time monitoring within a Kubernetes infrastructure.

---

## ✨ Key Features & Tools
* **CI/CD Pipeline:** Automated build and deployment using **GitHub Actions**.
* **GitOps:** Continuous Delivery managed by **ArgoCD** for sync-state configurations.
* **Monitoring & Observability:** Full stack monitoring using **Prometheus** & **Grafana**.
* **Logging:** Centralized log management with **Fluent Bit**&**Cloudwatch**.
* **Policy Management:** Enforcing security best practices using **Kyverno** policies.
* **Infrastructure:** Orchestrated on **Kubernetes (K8s)** with automated backups.

---

## 🏗️ Project Structure
Here is a breakdown of the repository organization:

| Folder / File | Description |
| :--- | :--- |
| 📂 `frontend/` | React/Web frontend source code and Dockerfile. |
| 📂 `backend/` | Python/Flask API source code and requirements. |
| 📂 `k8s/` | Kubernetes manifests (Deployments, Services, Ingress, and Backup CronJobs). |
| 📂 `.github/workflows` | Contains CI/CD pipelines (`backend-ci.yml`, `frontend-ci.yml`). |


---

## 🛠️ Tech Stack
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/github%20actions-%232088FF.svg?style=for-the-badge&logo=githubactions&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white)
![ArgoCD](https://img.shields.io/badge/argo%20cd-ef7b4d?style=for-the-badge&logo=argo-cd&logoColor=white)
![SonarCloud](https://img.shields.io/badge/SonarCloud-F3702A?style=for-the-badge&logo=sonarcloud&logoColor=white)
![Slack](https://img.shields.io/badge/Slack-4A154B?style=for-the-badge&logo=slack&logoColor=white)

---

---

## 🛠️ Prerequisites & Setup
To successfully deploy this project, please ensure you have completed the following steps:

### 1️⃣ General Requirements
- [ ] **GitHub Account:** For hosting the repository and executing **GitHub Actions** workflows.
- [ ] **AWS Account:** Active account with an **IAM User** (Programmatic access) having permissions to manage EKS, ECR, and VPC.
- [ ] **Infrastructure Ready:** The AWS Infrastructure (VPC, EKS Cluster, and Bastion Host) must be **fully deployed and running** before triggering the pipeline.
     The infrastructure is fully automated. You can provision it using the dedicated repository: [🚀 Infrastructure Repo (IaC)](https://github.com/Nabilaebrahim/iac-actions)
    * **Provisioning Pipeline:** Uses **Terraform** to deploy VPC, EKS Cluster, and Bastion Host.
    * **Auto-Configuration:** Uses **Ansible** via GitHub Actions to install:
        * **ArgoCD:** For GitOps CD.
        * **Monitoring:** Prometheus & Grafana stack.
        * **Security:** Kyverno policies.
        * **Logging:** Fluent Bit for CloudWatch integration.
    * **SSH Tunneling:** The pipeline automatically sets up a secure SSH tunnel via the Bastion host to communicate with the private EKS cluster.
- [ ] **SonarCloud:** An active account on [SonarCloud](https://sonarcloud.io/) for static code quality analysis.
- [ ] **Slack Integration:**  A dedicated Slack Workspace and Channel.
    * A **Slack App** with **Incoming Webhooks** enabled.
    * The Webhook URL copied for GitHub Secrets.

---

### 2️⃣ 🔐 GitHub Secrets Configuration
Navigate to `Settings > Secrets and variables > Actions` and add the following as **New repository secret**:

| Secret Name | Description |
| :--- | :--- |
| `AWS_ACCESS_KEY_ID` | AWS Access Key for your IAM User |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Key for your IAM User |
| `BASTION_SSH_KEY` | Private SSH Key to access the Bastion host |
| `SLACK_WEBHOOK_URL` | The Webhook URL from your Slack App |
| `SONAR_TOKEN` | Authentication token generated from SonarCloud |

---

### 3️⃣ 📂 GitHub Variables Configuration
In the same section, under the **Variables** tab, add the following as **New repository variable**:

| Variable Name | Description |
| :--- | :--- |
| `ECR_REPOSITORY` | Your Amazon ECR Repository name |
| `EKS_CLUSTER_NAME` | The name of your running EKS Cluster |
| `AWS_ACCOUNT_ID` | Your 12-digit AWS Account ID |
| `AWS_REGION` | The AWS Region (e.g., `us-east-1`) |

---


---

## 📂 Project Structure & Directories
Click on any folder to see the details of its files:

* [📂 backend](#-backend) - Backend source code and Docker configurations.
* [📂 frontend](#-frontend) - Frontend source code and Docker configurations.
* [📂 k8s](#-k8s) - Kubernetes manifests and policies.
* [📂 .github/workflows](#-githubworkflows) - CI/CD pipeline definitions.

---

## 📁 Detailed File Breakdown


### 📂 backend
The server-side logic of the application.
* **`app.py`**: The main entry point for the Flask/Python API.
* **`Dockerfile`**: Instructions to containerize the backend application.
* **`requirements.txt`**: List of dependencies needed for the Python environment.

---

### 📂 frontend
The user interface of the application.
* **`src/`**: Contains React components like `App.js` and `index.js`.
* **`public/`**: Static files like `index.html`.
* **`Dockerfile`**: Instructions to build the frontend image using Nginx.
* **`package.json`**: Project metadata and dependencies.

---

### 📂 k8s
Kubernetes resources to orchestrate the application.
* **`backend.yaml`**: Deployment and Service for the backend.
* **`frontend.yaml`**: Deployment and Service for the frontend.
* **`db.yaml`**: Database configurations (PostgreSQL).
    * 🗄️ **StatefulSet Architecture**: 
        * **Type**: Uses **StatefulSet** (instead of Deployment) to ensure stable network identifiers and persistent storage for the database pods.
        * **Replicas**: Configured with **2 replicas** for high availability.
        * **Storage**: Uses `volumeClaimTemplates` with **gp2** storage class to dynamically provision **1Gi** of persistent storage for each pod.
        * **Headless Service**: Includes a ClusterIP `None` service to allow direct communication with the individual database instances.
        * **Resources**: Resource limits and requests are set to manage CPU and memory usage efficiently.
* **`ingress.yaml`**: Rules for external access to the application.
    * 🌐 **Traffic Routing Control**: 
        * **Ingress Class**: Uses **Nginx** as the ingress controller to manage incoming traffic.
        * **Routing Rules**: 
            * `/` (Root): Routes all web traffic to the **Frontend** service on port `80`.
            * `/api`: Routes all API calls to the **Backend** service on port `5000`.
        * **Purpose**: Provides a single entry point for the entire 3-tier application, ensuring organized traffic flow between the user and the services.
* **`kyverno-policy.yaml`**: Security policies for the cluster.
    * 🛡️ **Image Signature Verification**: This policy inspects all **Pods** to verify they are running authorized images. It uses **Cosign** with a public key to check digital signatures, ensuring only trusted code is deployed (currently in `Audit` mode).
* **`backup-cronjob.yaml`**: Automated backup tasks.
    * 💾 **Scheduled Database Backup**: 
        * **Schedule**: Runs automatically every day at **12:00 AM (Midnight)**.
        * **Process**: It performs a `pg_dump` for the PostgreSQL databases (`db-0` and `db-1`), merges the backups, and securely uploads them to an **AWS S3 Bucket**.
        * **Security**: Uses AWS credentials stored in Kubernetes secrets to authenticate with S3.
        * **Retention**: Files are named with the current date (e.g., `full-backup-2024-05-20.sql`) for easy tracking.

---
### 📂 .github/workflows
This folder contains the GitHub Actions workflows for continuous integration.
* **`backend-ci.yml`**: Automates the **CI/CD pipeline** for the backend service.
    * 🔄 **Pipeline Flow Step-by-Step**:
        1. **Code Quality (SonarCloud)**: 🛡️ Scans the Python code for bugs, vulnerabilities, and code smells to ensure high-quality standards before building.
        2. **AWS Authentication**: 🔑 Securely logs into **AWS** using IAM credentials to interact with ECR and Secrets Manager.
        3. **Build & Push (Amazon ECR)**: 🐳 Builds the Docker image and pushes it to **Amazon Elastic Container Registry (ECR)** with a unique tag (Commit SHA).
        4. **Security Signing (Cosign & AWS Secrets Manager)**: 🔒 
            * Fetches a private signing key securely from **AWS Secrets Manager**.
            * Uses **Cosign** to digitally sign the Docker image. This ensures the image hasn't been tampered with (linked to our Kyverno policy).
        5. **GitOps Manifest Update**: 📝 Automatically updates the `backend.yaml` file in the `k8s/` folder with the new image tag.
        6. **ArgoCD Sync Trigger**: 🚀 By pushing the updated YAML back to GitHub, **ArgoCD** detects the change and automatically synchronizes (deploys) the new version to the EKS cluster.
* **`frontend-ci.yml`**: Automates the **CI/CD pipeline** for the React frontend application.
    * 🔄 **Pipeline Flow Step-by-Step**:
        1. **Static Analysis (SonarCloud)**: 🔍 Analyzes the frontend code to detect bugs and ensure maintainability.
        2. **AWS Authentication**: 🔑 Securely logs into **AWS** using repository secrets.
        3. **Containerization (Amazon ECR)**: 🐳 Builds the React production image using Docker and pushes it to **ECR** with the tag `frontend-SHA`.
        4. **Image Verification (Cosign)**: ✍️ 
            * Retrieves the signing key from **AWS Secrets Manager**.
            * Digitally signs the frontend image to guarantee security and compliance with the cluster policies.
        5. **Manifest Automation**: ⚙️ Updates the `frontend.yaml` in the `k8s/` folder with the latest image tag and AWS Account ID.
        6. **GitOps Trigger**: 🤖 Pushes the changes to the main branch, allowing **ArgoCD** to automatically sync the new frontend version.
        7. **Slack Notification**: 💬 Sends a real-time status report (Success/Failure) to the dedicated **Slack Channel**, including the commit details and a direct link to the Action run.


## 🛡️ Cluster Management (Bastion Host Access)

Since the EKS cluster is in a private subnet, we manage it through a **Bastion Host** (Jump Server). Follow these steps to configure your environment:

---

#### 1. Access the Bastion Host
First, set the correct permissions for your private key and SSH into the instance:

```bash
chmod 400 <YOUR_PRIVATE_KEY>.pem
ssh -i "<YOUR_PRIVATE_KEY>.pem" ubuntu@<BASTION_IP>
```
#### 2. Install kubectl
Download and install the Kubernetes command-line tool:

```bash
# Download the binary
curl -O https://s3.us-east-1.amazonaws.com/amazon-eks/1.31.2/2024-11-15/bin/linux/amd64/kubectl

# Make it executable and move to bin folder
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/
```
#### 3. Configure AWS Access & Link Cluster
Now, you need to authorize the Bastion host and link it to your EKS cluster:

```bash
# Provide your Access Key, Secret Key, and Region
  aws configure

# Update Kubeconfig to link the Bastion to your cluster
  aws eks update-kubeconfig --name <YOUR_CLUSTER_NAME> --region <YOUR_REGION>
```

#### 4. Verify Connection
Confirm that the Bastion host can successfully communicate with the EKS nodes:

```bash
  kubectl get nodes
```

## 🚀 Deployment Process

To deploy the application, you need to push your changes to the repository. This action will automatically trigger the CI/CD pipelines for both the Frontend and Backend.

---

### 1. Push Changes to GitHub
Follow these standard Git commands to update your code:

```bash
# Add all changes
git add .

# Check the status (Optional)
git status

# Commit your changes
git commit -m "Add: New features or fixes"

# Push to the main branch
git push origin main
```

### 2. CI/CD Pipeline Triggers
Once the push is successful, GitHub Actions will automatically trigger the following workflows:

* **Backend Pipeline:** `backend-ci.yml` starts building and deploying the backend services.
* **Frontend Pipeline:** `frontend-ci.yml` starts building and deploying the frontend interface.

> **💡 Tip:** You can monitor the progress of these pipelines in the **"Actions"** tab of your GitHub repository.


## ☸️ ArgoCD Access & Credentials

After the deployment, you can access the ArgoCD dashboard to monitor your applications. Use the following credentials to log in:

---

### 1. Login Details
* **Username:** `admin`
* **Password:** Use the command in the next step to retrieve it.

---

### 2. Retrieve Initial Admin Password
Run the following command in your terminal to get the decoded password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 3.Access ArgoCD via LoadBalancer**
To access the ArgoCD UI from your browser, you need to change the service type to `LoadBalancer` and get the external URL:

   * **A. Change Service Type to LoadBalancer**
     Run this command to expose the ArgoCD server:

     Run this command to expose the ArgoCD server:
     ```bash
     kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
     ```

   * **B. Get the External URL**
     Now, run this command to find the EXTERNAL-IP (the URL):

     ```bash
     kubectl get svc argocd-server -n argocd
     ```
> **💡 Note:** Note: Copy the address under the EXTERNAL-IP column and paste it into your browser to open the login page.



<img width="1920" height="937" alt="argo" src="https://github.com/user-attachments/assets/23ad051a-50e6-420a-b71f-1a1be7e7899c" />

<img width="1920" height="937" alt="argo2" src="https://github.com/user-attachments/assets/e783cef0-87f6-4fce-951d-82a33052028a" />

<img width="1920" height="923" alt="argo3" src="https://github.com/user-attachments/assets/b8c0d1f0-d5f5-4fe5-aae2-54e8048e6f60" />


## 📊 Monitoring with Grafana

Grafana is used to visualize the metrics collected from the EKS cluster. Follow these steps to access the dashboard without using port-forwarding.

---

### 1. Access Grafana via LoadBalancer
To get an external URL for Grafana, we need to change the service type:

   * **A. Change Service Type to LoadBalancer**
     Run this command to expose Grafana:
     ```bash
     kubectl patch svc prometheus-grafana -n monitoring -p '{"spec": {"type": "LoadBalancer"}}'
     ```

   * **B. Get the External URL**
     Run this command to find the **EXTERNAL-IP**:
     ```bash
     kubectl get svc -n monitoring prometheus-grafana
     ```
     > **Note:** Copy the address (e.g., `ad0962913069a463ea...elb.amazonaws.com`) and paste it into your browser.

---

### 2. Login Credentials
Once the page opens, use the following credentials:

* **Username:** `admin`
* **Password:** Retrieve the decoded password by running:

```bash
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d
```


<img width="1920" height="907" alt="prom" src="https://github.com/user-attachments/assets/ad7756e1-2f73-4686-99b3-1283da131b3e" />


## 💬 Slack Notifications

We have integrated Slack to receive real-time updates about the CI/CD pipeline status. This ensures that the team is immediately notified if a build succeeds or fails.

---

### 1. Slack Integration
The pipeline is configured to send notifications to a dedicated Slack channel using Webhooks.


* **Frontend Pipeline:** Sends status updates for every frontend build and deploy.
  

---

### 2. Notification Preview
Here is how the notifications appear in the Slack channel after a successful pipeline run:


> **💡 Note:** Each notification includes a link to the specific GitHub Action run for easy debugging.


<img width="1920" height="910" alt="slack" src="https://github.com/user-attachments/assets/63998af3-6e39-40c0-8f5f-9fd71a016aa5" />

