pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        DOCKER_IMAGE = "hanumath/n8n-task"
        KUBE_CONTEXT = "kind-devops-task"
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
                    dir('docker/images/n8n-task') {
                        sh "docker build -t $DOCKER_IMAGE:latest ."
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/']) {
                    sh "docker push $DOCKER_IMAGE:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl config use-context ${KUBE_CONTEXT}
                kubectl delete deployment n8n-task-deployment --ignore-not-found=true
                kubectl apply -f k8s/
                '''
            }
        }
    }

