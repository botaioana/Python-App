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
                sshagent([SSH_CREDENTIALS_ID]) {
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
                            CONTAINER_ID=\$(docker ps -q --filter "name=python-app")
                            if [ ! -z "\$CONTAINER_ID" ]; then
                                docker stop \$CONTAINER_ID && docker rm \$CONTAINER_ID
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
                        ssh -o StrictHostKeyChecking=no ${DOCKER_INSTANCE} 'docker run -d --name python-app -p 8080:8000 python-app:${BUILD_ID}'
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
