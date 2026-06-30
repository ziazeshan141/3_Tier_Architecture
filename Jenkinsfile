pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-2"
        AWS_ACCOUNT_ID = "047385030300"
        CLUSTER_NAME = "eks-cluster"

        FRONTEND_REPO = "047385030300.dkr.ecr.us-east-2.amazonaws.com/frontend"
        BACKEND_REPO  = "047385030300.dkr.ecr.us-east-2.amazonaws.com/backend"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Frontend') {
            steps {
                sh '''
                docker build -t frontend ./frontend
                '''
            }
        }

        stage('Build Backend') {
            steps {
                sh '''
                docker build -t backend ./backend
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Images') {
            steps {
                sh '''
                docker tag frontend:latest $FRONTEND_REPO:latest
                docker tag backend:latest $BACKEND_REPO:latest
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                docker push $FRONTEND_REPO:latest
                docker push $BACKEND_REPO:latest
                '''
            }
        }

        stage('Deploy to Amazon EKS') {
            steps {
                sh '''
                aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

                kubectl apply -f k8s/

                kubectl rollout restart deployment frontend
                kubectl rollout restart deployment backend

                kubectl rollout status deployment/frontend
                kubectl rollout status deployment/backend
                '''
            }
        }
    }

    post {
        success {
            echo 'Application deployed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }

        always {
            sh 'docker image prune -f || true'
        }
    }
}