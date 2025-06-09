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
        stage('Deploy to EC2 via SCP and SSH') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        echo "📦 Copying docker-compose.yml to EC2"
                        scp -o StrictHostKeyChecking=no -i "$SSH_KEY" "./docker_compose.yml" ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}/docker_compose.yml

                        echo "🚀 Deploying on remote EC2"
                        ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" ${DEPLOY_USER}@${DEPLOY_HOST} '
                            mkdir -p ${DEPLOY_PATH}
                            cd ${DEPLOY_PATH}
                            docker-compose pull
                            docker-compose up -d
                            echo "🔍 Running Containers:"
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
