pipeline {
    agent any

    environment {
        ECR_REPO = '992382545251.dkr.ecr.us-east-1.amazonaws.com/yuvaly-cicd'
        IMAGE_TAG = ''
        PROD_HOST = 'ec2-user@13.218.174.152'  
        PROD_PORT = 80                        
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

        stage('Set Image Tag') {
            steps {
                script {
                    if (env.CHANGE_ID) {
                        IMAGE_TAG = "pr-${env.CHANGE_ID}-${env.BUILD_NUMBER}"
                        IS_PR = true
                    } else if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master') {
                        IMAGE_TAG = "latest-${env.BUILD_NUMBER}" 
                        IS_PR = false
                    } else {
                        error "Branch not handled: ${env.BRANCH_NAME}"
                    }
                }
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_REPO
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

        stage('Test') {
            steps {
                sh '''
                    python -m unittest discover -s tests -v
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

        stage('Deploy to Production') {
            when { expression { return !IS_PR } }
            steps {
                sh """
                    ssh $PROD_HOST '
                    docker pull $ECR_REPO:$IMAGE_TAG
                    docker stop app || true
                    docker rm app || true
                    docker run -d --name app -p $PROD_PORT:80 $ECR_REPO:$IMAGE_TAG
                    '
                """
            }
        }

        stage('Health Check') {
            when { expression { return !IS_PR } } 
            steps {
                sh '''
                    python -m venv .venv && . .venv/bin/activate
                    pip install -r requirements.txt
                    python api.py &
                    pid=$!
                    for i in {1..5}; do
                        curl -fsS http://localhost:5000/health && kill $pid && exit 0
                        sleep 5
                    done
                    kill $pid
                    exit 1
                '''
            }
        }
    }

    post {
        always {
            junit 'tests/**/*.xml'  
        }
    }
}

