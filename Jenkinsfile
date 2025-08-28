pipeline {
    agent any

    environment {
        ECR_REPO = '992382545251.dkr.ecr.us-east-1.amazonaws.com/yuvaly-cicd'
        IMAGE_TAG = "latest-${env.BUILD_NUMBER}"
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

        stage('Install Python') {
            steps {
                dir('app') {
                    sh '''
                        apt-get update
                        apt-get install -y python3 python3-venv python3-pip
                        if [ ! -f /usr/bin/python ]; then
                            ln -s /usr/bin/python3 /usr/bin/python
                        fi
                        python -m venv .venv
                        . .venv/bin/activate
                        pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Run Unit/Integration Tests') {
            steps {
                dir('app') {
                    sh '''
                        . .venv/bin/activate
                        python -m unittest discover -s tests -v > results.xml || echo "No tests found"
                    '''
                }
                junit 'app/results.xml'
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
                script {
                    if (!IMAGE_TAG?.trim()) {
                        error "IMAGE_TAG is empty! Cannot build Docker image."
                    }
                }
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

        stage('Deploy to Production') {
            steps {
                sh '''
                    docker rm -f calculator || true
                    docker run -d -p 5000:5000 --name calculator $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Check Container Logs') {
            steps {
                sh '''
                    echo "=== Calculator logs ==="
                    docker logs calculator || true
                '''
            }
        }

        stage('Health Check') {
            steps {
                script {
                    def retries = 10
                    def wait = 3
                    def success = false

                    for (int i = 0; i < retries; i++) {
                        def status = sh(
                            script: "docker exec calculator curl -fsS http://127.0.0.1:5000/health || echo 'fail'",
                            returnStdout: true
                        ).trim()

                        if (status != "fail") {
                            echo "Health check passed!"
                            success = true
                            break
                        } else {
                            echo "Health check failed, retrying in ${wait}s..."
                            sleep(wait)
                        }
                    }

                    if (!success) {
                        error "Health check failed after ${retries*wait} seconds"
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker ps -a'
            sh 'docker images'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

