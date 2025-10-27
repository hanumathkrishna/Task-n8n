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

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image from docker/images/n8n/Dockerfile..."
                    sh """
                        cd docker/images/n8n
                        npm install
                        npm run build
                        cd ../../..
                        docker build -t ${DOCKER_IMAGE} -f docker/images/n8n/Dockerfile .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing image to Docker Hub..."
                    sh """
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying updated image to Kubernetes..."
                    sh """
                        kubectl set image deployment/n8n-deployment n8n-container=${DOCKER_IMAGE} --namespace=default || true
                        kubectl rollout restart deployment/n8n-deployment --namespace=default || true
                    """
                }
            }
        }
    }
}

