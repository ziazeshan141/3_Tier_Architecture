pipeline {
    agent any 

    environment {
        AWS_REGION = "us-east-2"
        CLUSTER_NAME = "eks-cluster"

        AWS_ACCOUNT_ID = "047385030300"
          
        FRONTEND_REPO = "047385030300.dkr.ecr.us-east-2.amazonaws.com/frontend"
        BACKEND_REPO = "047385030300.dkr.ecr.us-east-2.amazonaws.com/backend"
   }

   stages {

       stage('Checkout') {
            steps {
                checkout scm
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
               aws ecr get-login-password --region us-east-2 | \
               docker login --user
