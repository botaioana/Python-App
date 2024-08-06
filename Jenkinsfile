pipeline {
    agent any

    environment {
        SSH_CREDENTIALS_ID = '5ec6aa3f-9e35-4142-b621-0b0f8cfa0d1d'
        DOCKER_INSTANCE = 'ec2-user@ec2-35-158-140-230.eu-central-1.compute.amazonaws.com'
        REPO_URL = 'https://github.com/botaioana/Python-App.git'
        PROJECT_DIR = '/home/ec2-user/Python-App'
    }

    stages {
        stage('Clone Repository on Docker Instance') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_INSTANCE} <<EOF
                        echo "Checking if project directory exists..."
                        if [ ! -d "${PROJECT_DIR}" ]; then
                            echo "Directory not found. Cloning repository..."
                            git clone ${REPO_URL} ${PROJECT_DIR}
                        else
                            echo "Directory found. Pulling latest changes..."
                            cd ${PROJECT_DIR}
                            git pull origin main
                        fi
                        EOF
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_INSTANCE} <<EOF
                        echo "Navigating to project directory..."
                        cd ${PROJECT_DIR}
                        echo "Building Docker image..."
                        docker build -t python-app:${BUILD_ID} .
                        EOF
                    """
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DOCKER_INSTANCE} <<EOF
                        echo "Running Docker container..."
                        docker run -d -p 8080:8080 python-app:${BUILD_ID}
                        EOF
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
