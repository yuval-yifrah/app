pipeline {
    agent {
        docker {
            image 'docker:24.0-dind'
            args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
            user 'root'
        }
    }

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '992382545251.dkr.ecr.us-east-1.amazonaws.com/yuvaly-cicd'
    }

    stages {
        stage('Check Branch') {
            when {
                branch 'main'
            }
            steps {
                echo "Building on branch ${env.BRANCH_NAME}"
            }
        }

        stage('Login to ECR') {
            when {
                branch 'main'
            }
            steps {
                sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO'
            }
        }

        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    docker build -t yuvaly-cicd .
                    docker tag yuvaly-cicd:latest $ECR_REPO:latest
                '''
            }
        }

        stage('Push to ECR') {
            when {
                branch 'main'
            }
            steps {
                sh 'docker push $ECR_REPO:latest'
            }
        }
    }
}

