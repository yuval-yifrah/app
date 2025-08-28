pipeline {
    agent any

    environment {
        ECR_REPO = '992382545251.dkr.ecr.us-east-1.amazonaws.com/yuvaly-cicd'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                dir('app') {
                    git branch: 'yuval', url: 'https://github.com/yuval-yifrah/app.git'
                }
            }
        }

        stage('Checkout Platform Repo') {
            steps {
                dir('platform') {
                    git branch: 'main', url: 'https://github.com/yuval-yifrah/platform.git'
                }
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 992382545251.dkr.ecr.us-east-1.amazonaws.com
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

