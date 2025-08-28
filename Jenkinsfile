pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '992382545251.dkr.ecr.us-east-1.amazonaws.com/yuvaly-cicd'
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git url: 'https://github.com/yuval-yifrah/app.git'
            }
        }

        stage('Checkout Dockerfile Repo') {
            steps {
                git url: 'https://github.com/yuval-yifrah/platform.git'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws --version
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $ECR_REPO:$IMAGE_TAG platform
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }
    }
}

