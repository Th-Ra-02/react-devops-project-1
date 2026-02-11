# React-Devops-Project-1 (Guvi-Project-1)
  Containerized react application with EKS deployment

* Project Overview:
  This project demonstrates a full DevOps pipeline for deploying a React application to AWS EKS using CodeBuild,ECR,and Kubernetes. The pipeline  includes building the Docker image, pushing it to ECR, deploying to EKS, and tracking logs in CloudWatch.

* Prerequisites: 
  AWS Account with permissions to create:
  EKS cluster
  ECR repository
  CodeBuild project
  Docker installed locally for building/testing
  kubectl installed (used in build pipeline)

* Architecture:
  Docker Image
  Base: nginx:alpine
  Application files: dist/ directory copied into /usr/share/nginx/html.
  AWS Services Used
  ECR: Stores Docker images.
  EKS: Hosts Kubernetes cluster.
  CodeBuild: CI/CD pipeline to build, tag, push Docker images, and deploy to EKS.
  CloudWatch: Tracks CodeBuild logs and application logs via EKS logging.

* IAM Roles:
  codebuild-brain-tasks-codebuild-service-role: Permissions to push Docker images and access EKS cluster.
  Node group IAM role for worker nodes.
  Kubernetes
  aws-auth.yml maps IAM roles to Kubernetes RBAC.
  k8s/ folder contains deployment manifests.

# Setup Instructions

1. Push Docker Image to ECR
   CodeBuild automatically builds and pushes the Docker image using the following buildspec.yml steps:
   build:
   commands:
    - docker build -t $ECR_REPO .
    - docker tag $ECR_REPO:latest $ECR_ACCOUNT.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG

* If running manually:
  docker build -t brain-tasks-app .
  docker tag brain-tasks-app:latest <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/brain-tasks-app:latest
  docker push <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/brain-tasks-app:latest

2. Configure EKS Access
   Update kubeconfig to access the cluster:
   aws eks update-kubeconfig --region <REGION> --name brain-eks

* Verify access:
  kubectl get ns

3. Deploy Application to EKS
   Deploy all manifests in k8s/:
   kubectl apply -f k8s/ --validate=false

*  Check pods:
   kubectl get pods -n <NAMESPACE>

4. IAM and RBAC
   Ensure aws-auth.yml is applied to map IAM roles to Kubernetes roles:
   kubectl apply -f aws-auth.yml

*  This allows CodeBuild to deploy to EKS and nodes to join the cluster.

5. Logging
   Build & Deploy Logs: Tracked in AWS CloudWatch under the CodeBuild project.
   Application Logs: Available via Kubernetes logging in CloudWatch or via kubectl logs <pod-name>.
