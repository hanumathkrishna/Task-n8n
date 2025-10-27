pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "hanumath/n8n-task:latest"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build and Compile n8n') {
            steps {
                script {
                    echo "Installing dependencies and building n8n..."
                    sh '''
                        corepack enable
                        corepack prepare pnpm@10.18.3 --activate
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
}

