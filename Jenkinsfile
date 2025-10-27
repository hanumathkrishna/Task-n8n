pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "hanumath/n8n-task:latest"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')  // Jenkins ID for Docker Hub login
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
                    // Build from repo root, specify path to Dockerfile
                    sh """
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
                    echo "Deploying image to Kubernetes..."
                    // optional - modify this for your K8s cluster or skip for now
                    sh """
                        kubectl set image deployment/n8n-deployment n8n-container=${DOCKER_IMAGE} --namespace=default || true
                        kubectl rollout restart deployment/n8n-deployment --namespace=default || true
                    """
                }
            }
        }
    }

}

