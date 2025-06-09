pipeline {
    agent any

    environment {
        FRONTEND_REPO = 'https://github.com/UdayKiranChilumula/Frontend.git'
        BACKEND_REPO  = 'https://github.com/UdayKiranChilumula/Backend.git'

        FRONTEND_IMAGE = 'estate-frontend'
        BACKEND_IMAGE  = 'estate-backend'

        DEPLOY_USER = 'ec2-user'
        DEPLOY_HOST = '<EC2-B-PUBLIC-IP>'
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
                    """
                }
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh """
                        docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push udaykiranchilumula/${FRONTEND_IMAGE}
                        docker push udaykiranchilumula/${BACKEND_IMAGE}
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
