pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "lokanath467/pharmacy-app-ui"
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"

        CONTAINER_NAME = "pharmacy-ui"
        APP_PORT = "3000"
        HOST_PORT = "3000"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'lokanath467',
                    passwordVariable: '@467Lokanath'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh """
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker push ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                    docker pull ${DOCKER_IMAGE}:latest
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${HOST_PORT}:${APP_PORT} \
                        --restart unless-stopped \
                        ${DOCKER_IMAGE}:latest
                """
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