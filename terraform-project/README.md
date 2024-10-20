# **Terraform AWS Infrastructure Deployment**

This repository contains Terraform code to deploy a **task-listing application** on AWS using **Elastic Beanstalk, S3, RDS, ECR, and IAM roles**. 

The project also includes a **GitHub Actions pipeline** that validates and ensures the Terraform configuration is correct whenever a pull request (PR) is made and merged.

---

## **Infrastructure Overview**

This Terraform configuration provisions the following AWS resources:

- **Elastic Beanstalk Application and Environment**:  
  Hosts the Docker-based task-listing app.  
  - **Environment Variables**: Configures DB connection details (host, user, password).

- **IAM Roles and Policies**:  
  Allows EC2 instances in the Beanstalk environment to interact with ECR and other AWS resources.  

- **ECR (Elastic Container Registry)**:  
  Stores Docker images for deployment.  

- **S3 Buckets**:  
  - **Backend**: Stores the Terraform state file.  
  - **App Versions**: Stores app versions for Elastic Beanstalk.

- **RDS (PostgreSQL Database)**:  
  Stores the data for the task-listing app.

---

## **Terraform Backend Configuration**

The Terraform state is stored remotely in an **S3 bucket** to enable collaboration and state locking:

\`\`\`hcl
backend "s3" {
  bucket = "cl-terraform"
  key    = "terraform.tfstate"
  region = "eu-west-2"
}
\`\`\`

---

## **Pre-requisites**

Ensure you have the following:

- Terraform installed: [Download Terraform](https://www.terraform.io/downloads)
- AWS credentials configured (via \`aws configure\` or environment variables)
- Access to an AWS account with permissions for:
  - Elastic Beanstalk
  - RDS
  - IAM roles and policies
  - ECR
  - S3

---

## **How to Deploy the Infrastructure**

1. **Clone the Repository:**

   \`\`\`bash
   git clone <repo-url>
   cd <repo-folder>
   \`\`\`

2. **Initialize Terraform:**

   \`\`\`bash
   terraform init
   \`\`\`

3. **Validate the Configuration:**

   \`\`\`bash
   terraform validate
   \`\`\`

4. **Apply the Configuration:**

   \`\`\`bash
   terraform apply
   \`\`\`

   Confirm with \`yes\` when prompted.

5. **Destroy the Resources (when needed):**

   \`\`\`bash
   terraform destroy
   \`\`\`

---

## **GitHub Actions Pipeline for Terraform**

This repository uses a **GitHub Actions pipeline** to automate Terraform validation and application upon pull requests. Below is a summary of the pipeline’s workflow:

### **Pipeline Flow**:

1. **On Pull Request (PR)**:
   - The pipeline will:
     - Run \`terraform fmt\` to ensure the code is properly formatted.
     - Run \`terraform validate\` to check for any syntax errors.
     - Run a \`terraform plan\` to generate an execution plan.

2. **On Merge to Main Branch**:
   - After the PR is merged, the pipeline:
     - Runs \`terraform apply -auto-approve\` to apply the changes automatically.

---

### **GitHub Workflow File (\`.github/workflows/terraform.yml\`):**

\`\`\`yaml
name: Terraform CI/CD Pipeline

on:
  pull_request:
    branches:
      - main
  push:
    branches:
      - main

jobs:
  terraform:
    name: Terraform Validation and Deployment
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.5.6

      - name: Terraform Format Check
        run: terraform fmt -check -recursive

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan -out=tfplan

  apply:
    name: Terraform Apply on Merge
    needs: terraform
    runs-on: ubuntu-latest
    if: github.event_name == 'push'

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.5.6

      - name: Terraform Apply
        run: terraform apply -auto-approve tfplan
\`\`\`

---

## **Best Practices**

- **State Locking**: Ensure only one person makes changes to the Terraform state at a time by enabling state locking with the **S3 backend**.
- **Secrets Management**: Store sensitive data (e.g., DB password) in **AWS Secrets Manager** instead of hardcoding in Terraform.
- **Review PRs**: Validate all PRs to avoid infrastructure misconfigurations before merging.

---

## **Conclusion**

This Terraform setup provisions a scalable infrastructure on AWS to host a Docker-based application using Elastic Beanstalk, RDS, and more. The **GitHub Actions pipeline** ensures that all Terraform code is validated before deployment, maintaining the integrity of the infrastructure.

Feel free to contribute by submitting PRs!

---

## **Contact**

For questions or issues, contact the maintainer at [lachlan.hooker@example.com].
"""
