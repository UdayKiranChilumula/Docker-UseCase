pipeline {
    agent any

    environment {
        FRONTEND_REPO = 'https://github.com/UdayKiranChilumula/Frontend.git'
        BACKEND_REPO  = 'https://github.com/UdayKiranChilumula/Backend.git'

        FRONTEND_IMAGE = 'estate-frontend'
        BACKEND_IMAGE  = 'estate-backend'

        DEPLOY_USER = 'ec2-user'
        DEPLOY_HOST = '35.171.26.232'
        DEPLOY_PATH = '/home/ec2-user/estate-deployment'
    }

    stages {
        stage('Clone Repos') {
            steps {
                dir('frontend') {
                    git url: "${FRONTEND_REPO}", branch: 'main'
                }
                dir('backend') {
                    git url: "${BACKEND_REPO}", branch: 'main'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh """
                        docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} .
                        docker tag ${FRONTEND_IMAGE}:${BUILD_NUMBER} udaykiranchilumula/${FRONTEND_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh """
                        docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} .
                        docker tag ${BACKEND_IMAGE}:${BUILD_NUMBER} udaykiranchilumula/${BACKEND_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push udaykiranchilumula/${FRONTEND_IMAGE}:${BUILD_NUMBER}
                        docker push udaykiranchilumula/${BACKEND_IMAGE}:${BUILD_NUMBER}
                    """
                }
            }
        }
        stage('Deploy to EC2 via SSH') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                sh """
                echo "🚀 SSH into EC2 and deploy"

                ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" ${DEPLOY_USER}@${DEPLOY_HOST} '
                    set -e
                    sudo yum install -y git
                    mkdir -p ${DEPLOY_PATH}
                    cd ${DEPLOY_PATH}

                    echo "📥 Cloning/Updating docker-compose repo..."
                    if [ ! -d ".git" ]; then
                        git clone https://github.com/UdayKiranChilumula/DockerComposeRepo.git . 
                    else
                        git pull origin main
                    fi

                    echo "📦 Pulling latest Docker images..."
                    docker-compose pull

                    echo "🔧 Starting containers..."
                    docker-compose up -d

                    echo "✅ Containers running:"
                    docker ps
                '
            """
        }
    }
}


    }

    post {
        always {
            echo '🧹 Cleaning up frontend/ and backend/ directories...'
            dir('frontend') {
                deleteDir()
            }
            dir('backend') {
                deleteDir()
            }
        }
    }
}
