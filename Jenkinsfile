pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "hanumath/n8n-task:latest"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        PNPM_STORE = "/var/lib/jenkins/.pnpm-store"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }

        stage('Prepare Environment') {
            steps {
                script {
                    echo "Setting up Node, Corepack, and pnpm..."
                    sh '''
                        mkdir -p ${PNPM_STORE}
                        export PNPM_STORE_PATH=${PNPM_STORE}
                        corepack enable
                        corepack prepare pnpm@10.18.3 --activate
                    '''
                }
            }
        }

        stage('Install Dependencies & Build n8n') {
            steps {
                script {
                    echo "Installing dependencies and building source..."
                    sh '''
                        export PNPM_STORE_PATH=${PNPM_STORE}
                        pnpm install --frozen-lockfile
                        pnpm build
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh '''
                        docker build -t ${DOCKER_IMAGE} -f docker/images/n8n/Dockerfile .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub..."
                    sh '''
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }
}

