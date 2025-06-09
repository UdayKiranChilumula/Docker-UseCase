pipeline {
    agent any

    environment {
        FRONTEND_REPO = 'https://github.com/UdayKiranChilumula/Frontend.git'
        BACKEND_REPO  = 'https://github.com/UdayKiranChilumula/Backend.git'

        FRONTEND_IMAGE = 'udaykiranchilumula/estate-frontend'
        BACKEND_IMAGE  = 'udaykiranchilumula/estate-backend'

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
                    sh "docker build -t ${FRONTEND_IMAGE}:latest ."
                }
               
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh "docker build -t ${BACKEND_IMAGE}:latest ."
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
