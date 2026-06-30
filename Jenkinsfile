pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    environment {
        AWS_REGION = "us-east-2"
        CLUSTER_NAME = "eks-cluster"

        AWS_ACCOUNT_ID = "047385030300"

        FRONTEND_REPO = "047385030300.dkr.ecr.us-east-2.amazonaws.com/frontend"
        BACKEND_REPO = "047385030300.dkr.ecr.us-east-2.amazonaws.com/backend"

        SCANNER_HOME = tool 'SonarScanner'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=3-tier-app \
                    -Dsonar.projectName=3-tier-app \
                    -Dsonar.sources=frontend,backend \
                    -Dsonar.sourceEncoding=UTF-8
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Frontend') {
            steps {
                sh 'docker build -t frontend ./frontend'
            }
        }

        stage('Build Backend') {
            steps {
                sh 'docker build -t backend ./backend'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                docker tag frontend:latest $FRONTEND_REPO:latest
                docker tag backend:latest $BACKEND_REPO:latest

                docker push $FRONTEND_REPO:latest
                docker push $BACKEND_REPO:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

                kubectl apply -f k8s/

                kubectl rollout restart deployment frontend
                kubectl rollout restart deployment backend
                '''
            }
        }
    }

    post {
        success {
            echo "Application deployed successfully"
        }

        failure {
            echo "Pipeline failed"
        }
    }
}