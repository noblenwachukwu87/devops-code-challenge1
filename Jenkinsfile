pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "491865087343"
        AWS_REGION     = "us-east-1"
        ECR_BACKEND    = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/devops-challenge1-backend"
        ECR_FRONTEND   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/devops-challenge1-frontend"
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/noblenwachukwu87/devops-code-challenge1.git',
                    credentialsId: 'github-credentials'
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('backend') {
                    sh "docker build --no-cache --platform linux/amd64 -t ${ECR_BACKEND}:latest ."
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    sh "docker build --no-cache  --platform linux/amd64 -t ${ECR_FRONTEND}:latest ."
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                docker push ${ECR_BACKEND}:latest
                docker push ${ECR_FRONTEND}:latest
                """
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh """
                aws ecs update-service --cluster devops-challenge1-cluster --service devops-challenge1-backend-service --force-new-deployment --region ${AWS_REGION}
                aws ecs update-service --cluster devops-challenge1-cluster --service devops-challenge1-frontend-service --force-new-deployment --region ${AWS_REGION}
                """
            }
        }
    }
}
