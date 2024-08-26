pipeline {
    agent any

    environment {
        SSH_CREDENTIALS_ID = '5ec6aa3f-9e35-4142-b621-0b0f8cfa0d1d'
        DOCKER_INSTANCE = 'ec2-user@ec2-35-158-140-230.eu-central-1.compute.amazonaws.com'
        REPO_URL = 'https://github.com/botaioana/Python-App.git'
        PROJECT_DIR = '/home/ec2-user/Python-App'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Clone Repository on Docker Instance') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {//if the project doesn't exist at the location, it clones it, else it pulls it
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_INSTANCE} 'if [ ! -d "${PROJECT_DIR}" ]; then git clone ${REPO_URL} ${PROJECT_DIR}; else cd ${PROJECT_DIR} && git pull origin main; fi'
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_INSTANCE} 'cd ${PROJECT_DIR} && docker build -t python-app:${BUILD_ID} .'
                    """
                }
            }
        }

        stage('Stop Previous Container') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_INSTANCE} '
                            CONTAINER_ID=\$(docker ps -a -q --filter "name=python-app")
                            if [ ! -z "\$CONTAINER_ID" ]; then
                                echo "Stopping and removing existing container \$CONTAINER_ID"
                                docker stop \$CONTAINER_ID && docker rm \$CONTAINER_ID
                                sleep 15  # Adding a longer delay to ensure the port is freed
                            else
                                echo "No existing container found"
                            fi
                        '
                    """
                }
            }
        }
        stage('Stop Previous Image') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh"""
                        ssh -o StrictHostKeyChecking=no ${DOCKER_INSTANCE} '
                        IMAGE_ID=\$(docker image -q --filter "name=python-app")
                        if [ ! -z "\$IMAGE_ID"]; then
                            echo "Stopping and removing existing image \$IMAGE_ID"
                            docker rmi \$IMAGE_ID
                            sleep 15
                        else
                            echo "No existing image found"
                        fi
                    '
                    """
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_INSTANCE} '
                            while lsof -i:8080; do
                                echo "Port 8080 is still in use. Waiting..."
                                sleep 5  # Increased sleep duration to give more time for the port to be freed
                            done
                            docker run -d --name python-app -p 8080:8000 python-app:${BUILD_ID}
                        '
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
