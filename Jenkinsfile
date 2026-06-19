def IMAGE_TAG = ''

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

        stage('Set Image Tag') {
            steps {
                script {
                    if (env.TAG_NAME) {
                        IMAGE_TAG = env.TAG_NAME
                    } else if (env.GIT_BRANCH == 'origin/main' || env.GIT_BANCH == 'main') {
                        IMAGE_TAG = 'latest'
                    } else if (env.GIT_BANCH == 'origin/develop' || env.GIT_BANCH == 'develop') {
                        IMAGE_TAG = "develop-${env.GIT_COMMIT[0..6]}"
                    } else {
                        IMAGE_TAG = "branch-${env.GIT_COMMIT[0..6]}"
                    }
                    echo "Image tag: ${IMAGE_TAG}"
                }
            }
        }

        stage('Build') {
            steps {
                sh "podman build -t ${IMAGE_NAME}:${IMAGE_TAG} ./app"
            }
        }

        stage('Push') {
            steps {
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | podman login docker.io \
                    -u $DOCKERHUB_CREDENTIALS_USR \
                    --password-stdin
                '''
                sh "podman push ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
    }
}