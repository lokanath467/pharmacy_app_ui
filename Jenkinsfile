pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "lokanath467/pharmacy-app-ui"
        DOCKER_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "pharmacy-ui"
        APP_PORT = "3000"
        HOST_PORT = "3000"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'sudo-password', variable: 'SUDO_PASS')]) {
                    sh '''
                        echo "$SUDO_PASS" | sudo -S docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        echo "$SUDO_PASS" | sudo -S docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    ),
                    string(credentialsId: 'sudo-password', variable: 'SUDO_PASS')
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | echo "$SUDO_PASS" | sudo -S docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([string(credentialsId: 'sudo-password', variable: 'SUDO_PASS')]) {
                    sh '''
                        echo "$SUDO_PASS" | sudo -S docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        echo "$SUDO_PASS" | sudo -S docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                withCredentials([string(credentialsId: 'sudo-password', variable: 'SUDO_PASS')]) {
                    sh '''
                        echo "$SUDO_PASS" | sudo -S docker pull ${DOCKER_IMAGE}:latest
                        echo "$SUDO_PASS" | sudo -S docker rm -f ${CONTAINER_NAME} || true
                        echo "$SUDO_PASS" | sudo -S docker run -d \
                            --name ${CONTAINER_NAME} \
                            -p ${HOST_PORT}:${APP_PORT} \
                            --restart unless-stopped \
                            ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Build, Push & Deployment Successful!"
        }
        failure {
            echo "Pipeline Failed!"
        }
        always {
            cleanWs()
        }
    }
}