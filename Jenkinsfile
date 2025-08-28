pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '992382545251.dkr.ecr.us-east-1.amazonaws.com/yuvaly-cicd'
        IMAGE_TAG = "latest"
        DOCKERFILE_REPO = '/var/jenkins_home/dockerfiles-repo' // <-- נתיב לרפו/תיקיית Dockerfile
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                checkout scm
            }
        }

        stage('Checkout Dockerfile Repo') {
            steps {
                git url: 'https://github.com/yuval-yifrah/dockerfiles-repo.git', branch: 'main', changelog: false
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
                sh """
                    docker build -t $ECR_REPO:$IMAGE_TAG $DOCKERFILE_REPO
                """
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

