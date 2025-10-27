pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "hanumath/n8n-task:latest"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        PNPM_STORE_PATH = '/var/lib/jenkins/.pnpm-store'
        PATH = "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Fetching latest source code..."
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

                        if ! command -v corepack >/dev/null 2>&1; then
                            echo "Installing Node.js and Corepack..."
                            curl -fsSL https://nodejs.org/dist/v20.19.0/node-v20.19.0-linux-x64.tar.xz -o node.tar.xz
                            sudo tar -xJf node.tar.xz -C /usr/local --strip-components=1
                            rm -f node.tar.xz
                        fi

                        export PATH=/usr/local/bin:$PATH
                        corepack enable
                        corepack prepare pnpm@10.18.3 --activate
                        echo "Node version: $(node -v)"
                        echo "PNPM version: $(pnpm -v)"
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
                    echo "Preparing /compiled folder for Docker build..."
                    sh '''
                        rm -rf compiled
                        mkdir -p compiled

                        # Collect all built package files (dist folders)
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
                    echo "Building Docker image..."
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

        stage('Deploy to Kubernetes (via Helm)') {
            steps {
                script {
                    echo "Deploying n8n to Kubernetes via Helm..."
                    sh """
                        helm upgrade --install n8n-task ./helm/n8n \
                            --set image.repository=${DOCKER_IMAGE.split(':')[0]} \
                            --set image.tag=${DOCKER_IMAGE.split(':')[1]} \
                            --namespace default --create-namespace
                    """
                }
            }
        }
    }
}

