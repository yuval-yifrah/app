pipeline {
    agent any

    environment {
        ECR_REPO = '992382545251.dkr.ecr.us-east-1.amazonaws.com/yuvaly-cicd'       // כתוב כאן את שם ה־ECR שלך
        IMAGE_TAG = 'latest'             // או כל tag שתרצה
    }

    stages {
        stage('Checkout Platform Repo') {
            steps {
                git branch: 'create', url: 'https://github.com/yuval-yifrah/platform.git'
            }
        }

        stage('Checkout App Repo') {
            steps {
                git branch: 'yuval', url: 'https://github.com/yuval-yifrah/app.git'
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
                    aws ecr get-login-password --region YOUR_AWS_REGION | docker login --username AWS --password-stdin YOUR_AWS_ACCOUNT_ID.dkr.ecr.YOUR_AWS_REGION.amazonaws.com
                    docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }
    }
}

