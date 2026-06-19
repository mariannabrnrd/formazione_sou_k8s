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
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | podman login docker.io \
                    -u $DOCKERHUB_CREDENTIALS_USR \
                    --password-stdin
                    podman push mariannabrnrd/flask-app-example:latest
                '''
            }
        }
    }
}