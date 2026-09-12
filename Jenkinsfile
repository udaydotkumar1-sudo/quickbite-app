pipeline {
    agent any

environment {
    DOCKER_HUB_REPO = 'udayadmin1/quickbite-frontend'
    APP_SERVER_IP   = '54.12.34.56'
    DOCKER_CREDS    = credentials('dockerhub-credentials')
}

    stages {
        stage('Stage 1 — Checkout') {
            steps {
                echo "Pulling latest source code from GitHub..."
                checkout scm
            }
        }

        stage('Stage 2 — Build') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKER_HUB_REPO}:build-${BUILD_NUMBER} ."
            }
        }

        stage('Stage 3 — Tag') {
            steps {
                echo "Tagging Docker image with build number and latest..."
                sh "docker tag ${DOCKER_HUB_REPO}:build-${BUILD_NUMBER} ${DOCKER_HUB_REPO}:latest"
            }
        }

        stage('Stage 4 — Push') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                sh '''
                    echo "$DOCKER_CREDS_PSW" | docker login -u "$DOCKER_CREDS_USR" --password-stdin
                    docker push ${DOCKER_HUB_REPO}:build-${BUILD_NUMBER}
                    docker push ${DOCKER_HUB_REPO}:latest
                '''
            }
        }

        stage('Stage 5 — Deploy') {
            steps {
                echo "Deploying container to Application Server..."
                sshagent(['app-server-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER_IP} '
                        docker pull ${DOCKER_HUB_REPO}:latest
                        docker stop quickbite-frontend || true
                        docker rm quickbite-frontend || true
                        docker run -d --name quickbite-frontend --restart always -p 80:80 ${DOCKER_HUB_REPO}:latest
                    '
                    """
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
