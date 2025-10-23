📝 InnovateMart Project Bedrock: Deployment & Architecture Guide
1. Project Overview & Deliverables

Git Repository Link: https://github.com/IJOBA007/innovatemart-bedrock

This project successfully deployed the retail-store-sample-app on a production-grade Amazon EKS cluster using Terraform (IaC).
All Core Requirements (3.1–3.4) were met.
The infrastructure was successfully verified and then destroyed to ensure fiscal responsibility.

2. Core Architecture Overview (3.1 & 3.2)

The infrastructure was provisioned using Terraform in the eu-west-1 (Ireland) region.

VPC: Created for high availability, with private subnets hosting the core infrastructure.

EKS Cluster: The Control Plane was deployed. A Managed Node Group of t3.medium instances was launched into the private subnets.

IAM Roles: Dedicated roles were provisioned (e.g., innovatemart-eks-cluster-role) following the least privilege principle.

Application: The app was deployed using in-cluster dependencies (MySQL, PostgreSQL, Redis, etc.) running as Pods alongside the microservices, fulfilling Core Requirement 3.2.

3. Developer Access & Instructions (3.3)

The read-only access for the development team is secured using IAM Roles for Kubernetes Access (aws-auth ConfigMap mapping).

The designated read-only user is innovatemart-dev (created via IaC) and is mapped to the Kubernetes system:viewers group.

Instructions for Read-Only Access

Step 1: Configure AWS CLI
The developer must first configure their AWS CLI profile using their unique innovatemart-dev Access and Secret keys:

aws configure


Step 2: Generate Kubeconfig
Run the following command to link the IAM identity to the cluster, granting read-only permissions:

aws eks update-kubeconfig --name innovatemart-eks --region eu-west-1


Step 3: Verification
The developer can verify access by running:

kubectl get pods


✅ Expected: Success
❌ Attempting:

kubectl delete pod <pod-name>


Will return: Error: forbidden

4. CI/CD and Automation (3.4)

The automation plan uses GitHub Actions with secure credential management via GitHub Secrets (e.g., AWS_ACCESS_KEY_ID).

The terraform.yml pipeline implements the following branching strategy:

Pull Requests (Feature Branches): Triggers terraform plan.

Merge to main: Triggers

terraform apply --auto-approve

5. Bonus Objectives Implementation (Extra Marks)
Advanced Networking & Security (4.2)

This objective was designed in code but not fully deployed to ensure a clean cleanup process.

Load Balancer & Ingress: The Ingress manifest was prepared with

kubernetes.io/ingress.class: "alb"


to instruct the AWS Load Balancer Controller to provision an Application Load Balancer (ALB).

HTTPS/SSL: The Ingress was annotated with

alb.ingress.kubernetes.io/ssl-certificate: <ACM ARN>


to enable HTTPS using an AWS Certificate Manager (ACM) certificate attached to the ALB.

Route 53: The process requires registering a placeholder domain (e.g., store.innovatemart.com) and creating an Alias Record pointing to the ALB’s DNS name.

Managed Persistence (4.1)

This objective was partially defined in IaC:

Terraform code was created to provision:

AWS RDS for PostgreSQL

AWS RDS for MySQL

Amazon DynamoDB table

Database credentials were securely managed via AWS Secrets Manager, avoiding plaintext storage in the Git repository.

Final integration would require an External Secrets Operator in EKS.
