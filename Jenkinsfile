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
                script {
                    def containerName = "calculator"
                    def image = "${ECR_REPO}:${IMAGE_TAG}"

                    // מחיקת קונטיינר ישן אם קיים
                    sh "docker rm -f ${containerName} || true"

                    // הרצת קונטיינר חדש עם network host
                    sh "docker run -d --name ${containerName} --network host ${image}"

                    // בדיקת בריאות עם retry
                    retry(5) {
                        sleep 6
                        def status = sh(script: 'curl -fsS http://localhost:5000/health || echo "fail"', returnStdout: true).trim()
                        if (status == "fail") {
                            error "Health check failed"
                        }
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

