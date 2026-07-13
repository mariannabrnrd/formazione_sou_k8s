def IMAGE_TAG = ''

pipeline {
    agent { label 'docker-agent' }

    parameters {
        choice(
            name: 'BRANCH',
            choices: ['main', 'develop'],
            description: 'Seleziona il branch da buildare'
        )
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        IMAGE_NAME = "mariannabrnrd/flask-app-example"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "${params.BRANCH}"]],
                        userRemoteConfigs: [[
                            url: 'https://github.com/mariannabrnrd/formazione_sou_k8s',
                        ]]
                    ])
                }
            }
        }

        stage('Set Image Tag') {
            steps {
                script {
                    if (env.TAG_NAME) {
                        IMAGE_TAG = env.TAG_NAME
                    } else if (params.BRANCH == 'main') {
                        IMAGE_TAG = 'latest'
                    } else if (params.BRANCH == 'develop') {
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