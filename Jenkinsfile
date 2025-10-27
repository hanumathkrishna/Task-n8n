pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "hanumath/n8n-task:latest"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        PNPM_STORE_PATH = '/var/lib/jenkins/.pnpm-store'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Environment') {
            steps {
                script {
                    echo "Setting up Node, Corepack, and pnpm..."
                    sh '''
                        mkdir -p $PNPM_STORE_PATH
                        export PNPM_STORE_PATH=$PNPM_STORE_PATH

                        # Make sure Node and Corepack are ready
                        if ! command -v corepack >/dev/null 2>&1; then
                            echo "Installing Node & Corepack..."
                            curl -fsSL https://nodejs.org/dist/v20.19.0/node-v20.19.0-linux-x64.tar.xz -o node.tar.xz
                            sudo tar -xJf node.tar.xz -C /usr/local --strip-components=1
                            rm -f node.tar.xz
                            sudo corepack enable
                        fi

                        corepack enable
                        corepack prepare pnpm@10.18.3 --activate
                        pnpm -v
                        node -v
                    '''
                }
            }
        }

        stage('Install Dependencies & Build n8n') {
            steps {
                script {
                    echo "Installing dependencies and building n8n source..."
                    sh '''
                        pnpm install --frozen-lockfile
                        pnpm build
                    '''
                }
            }
        }

        stage('Prepare Compiled Folder') {
            steps {
                script {
                    echo "Creating /compiled folder for Docker build..."
                    sh '''
                        rm -rf compiled
                        mkdir -p compiled

                        # Collect built packages
                        find packages -type d -name "dist" -exec rsync -a {}/ compiled/ \\;
                        echo "Compiled folder created successfully."
                        ls -la compiled | head -20
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image from Dockerfile..."
                    sh """
                        docker build -t ${DOCKER_IMAGE} -f docker/images/n8n/Dockerfile .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub..."
                    sh """
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }
}

