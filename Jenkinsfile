pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "dharani182/jenkins-demo"
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS = "dockerhub"
        SSH_CREDENTIALS = "vm-ssh-key"
        DEPLOY_SERVER = "15.252.8.71"
        CONTAINER_NAME = "jenkins-demo"
        APP_PORT = "8080"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'git-for-jk',
                    url: 'https://github.com/awspractice138-cloud/devops-project.git'
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    echo "===== Workspace ====="
                    pwd

                    echo "===== Files ====="
                    ls -la

                    echo "===== Docker Version ====="
                    docker --version
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker Image..."

                    docker build \
                        -t ${DOCKER_IMAGE}:${DOCKER_TAG} \
                        -t ${DOCKER_IMAGE}:latest .
                '''
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push dharani182/jenkins-demo:${DOCKER_TAG}
                        docker push dharani182/jenkins-demo:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {

                sshagent(credentials: ['vm-ssh-key']) {

                    sh '''
                        echo "Deploying to ${DEPLOY_SERVER}..."

                        ssh -o StrictHostKeyChecking=no \
                            ubuntu@${DEPLOY_SERVER} "
                                docker pull ${DOCKER_IMAGE}:latest

                                docker stop ${CONTAINER_NAME} || true

                                docker rm ${CONTAINER_NAME} || true

                                docker run -d \
                                    --name ${CONTAINER_NAME} \
                                    -p ${APP_PORT}:80 \
                                    --restart unless-stopped \
                                    ${DOCKER_IMAGE}:latest

                                docker ps
                            "
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {

                sh '''
                    echo "Waiting for application..."

                    sleep 10

                    echo "Checking application..."

                    curl -f http://${DEPLOY_SERVER}:${APP_PORT}/

                    echo "Application is UP!"
                '''
            }
        }
    }

    post {

        success {
            echo "======================================"
            echo " CI/CD PIPELINE SUCCESSFUL"
            echo " Build Number : ${BUILD_NUMBER}"
            echo " Docker Image : ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo " Application  : http://${DEPLOY_SERVER}:${APP_PORT}"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo " CI/CD PIPELINE FAILED"
            echo " Build Number : ${BUILD_NUMBER}"
            echo " Check Jenkins Console Output"
            echo "======================================"
        }

        always {
            sh '''
                echo "Cleaning local Docker images..."

                docker image prune -f || true
            '''
        }
    }
}


