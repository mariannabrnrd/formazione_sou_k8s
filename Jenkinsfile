pipeline {
    agent { label 'docker-agent' }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        IMAGE_NAME = "mariannabrnrd/flask-app-example"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'podman build -t ${IMAGE_NAME}:latest ./app'
            }
        }

        stage('Push') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | podman login docker.io -u $DOCKER_CREDENTIALS_USR --password-stdin'
                sh 'podman push ${IMAGE_NAME}:latest' 
            }
        }
    }
}