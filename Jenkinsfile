pipeline {
    agent any

    environment {
        SSH_CREDENTIALS_ID = '5ec6aa3f-9e35-4142-b621-0b0f8cfa0d1d' // Replace with your actual SSH credentials ID
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/botaioana/Python-App.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@ec2-35-158-140-230.eu-central-1.compute.amazonaws.com << EOF
                        cd /c/Users/Ioana/python-docker-example
                        docker build -t python-app:${BUILD_ID} .
                        EOF
                    '''
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@ec2-35-158-140-230.eu-central-1.compute.amazonaws.com << EOF
                        docker run -d -p 8080:8080 python-app:${BUILD_ID}
                        EOF
                    '''
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
