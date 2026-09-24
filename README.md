## Setup Instructions

### Prerequisites
- AWS CLI configured with credentials
- Terraform installed
- Docker installed
- Git

### 1. Clone and provision infrastructure
```bash
git clone https://github.com/noblenwachukwu87/devops-code-challenge1.git
cd devops-code-challenge1/terraform
terraform init
terraform apply
```

### 2. Build and push Docker images to ECR
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com

cd ../backend
docker build --platform linux/amd64 -t <account-id>.dkr.ecr.us-east-1.amazonaws.com/devops-challenge1-backend:latest .
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/devops-challenge1-backend:latest

cd ../frontend
docker build --platform linux/amd64 -t <account-id>.dkr.ecr.us-east-1.amazonaws.com/devops-challenge1-frontend:latest .
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/devops-challenge1-frontend:latest
```

### 3. Force ECS to deploy the new images
```bash
aws ecs update-service --cluster devops-challenge1-cluster --service devops-challenge1-backend-service --force-new-deployment
aws ecs update-service --cluster devops-challenge1-cluster --service devops-challenge1-frontend-service --force-new-deployment

### 4. Access Jenkins for CI/CD
Jenkins runs at `http://<jenkins-ec2-public-ip>:8080`. Credentials for GitHub and AWS are stored in Jenkins' credential store (`github-credentials`, `aws-access-key-id`, `aws-secret-access-key`).

## Challenges & Solutions

**Stale Docker image despite correct source code**
After confirming a config fix was committed and pushed to GitHub, the deployed frontend still exhibited the old behavior. Investigation traced this to Docker reusing a cached build layer during the image build, even though the underlying source file had changed — meaning the image tagged `:latest` in ECR didn't actually reflect the latest commit. Diagnosed by pulling the deployed image locally and grepping the compiled bundle for the stale reference, confirming a mismatch between source and artifact. Fixed by adding `--no-cache` to the Jenkinsfile's Docker build steps, ensuring every CI run rebuilds fully from source rather than trusting cached layers.

**Cross-architecture image incompatibility (ARM64 vs. x86_64)**
Docker images built locally on an Apple Silicon Mac defaulted to the ARM64 architecture, but AWS Fargate requires linux/amd64. Tasks failed to start with a `CannotPullContainerError` citing a platform mismatch, despite the image existing and being valid. Resolved by explicitly specifying `--platform linux/amd64` on every `docker build` command, both locally and in the CI pipeline — a necessary step for any team with a mix of Apple Silicon and x86 developer machines deploying to standard cloud infrastructure.
