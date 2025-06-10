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
        stage('Detect Changes') {
            steps {
                script {
                    def isFirstBuild = currentBuild.previousBuild == null
                    env.FRONTEND_CHANGED = isFirstBuild ? 'true' : 'false'
                    env.BACKEND_CHANGED = isFirstBuild ? 'true' : 'false'

                    if (!isFirstBuild) {
                        def changeLog = currentBuild.changeSets
                        for (changeSet in changeLog) {
                            for (entry in changeSet.items) {
                                for (file in entry.affectedFiles) {
                                    if (file.path.startsWith('frontend/')) {
                                        env.FRONTEND_CHANGED = 'true'
                                    }
                                    if (file.path.startsWith('backend/')) {
                                        env.BACKEND_CHANGED = 'true'
                                    }
                                }
                            }
                        }
                    }

                    echo "Frontend Changed: ${env.FRONTEND_CHANGED}"
                    echo "Backend Changed: ${env.BACKEND_CHANGED}"
                }
            }
        }

        stage('Clone Frontend') {
            when {
                expression { env.FRONTEND_CHANGED == 'true' }
            }
            steps {
                dir('frontend') {
                    git url: "${FRONTEND_REPO}", branch: 'main'
                }
            }
        }

        stage('Clone Backend') {
            when {
                expression { env.BACKEND_CHANGED == 'true' }
            }
            steps {
                dir('backend') {
                    git url: "${BACKEND_REPO}", branch: 'main'
                }
            }
        }

        stage('Build Frontend') {
            when {
                expression { env.FRONTEND_CHANGED == 'true' }
            }
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
            when {
                expression { env.BACKEND_CHANGED == 'true' }
            }
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
            when {
                anyOf {
                    expression { env.FRONTEND_CHANGED == 'true' }
                    expression { env.BACKEND_CHANGED == 'true' }
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        if [ "${FRONTEND_CHANGED}" = "true" ]; then
                            docker push udaykiranchilumula/${FRONTEND_IMAGE}:${BUILD_NUMBER}
                        fi
                        if [ "${BACKEND_CHANGED}" = "true" ]; then
                            docker push udaykiranchilumula/${BACKEND_IMAGE}:${BUILD_NUMBER}
                        fi
                    """
                }
            }
        }

        stage('Deploy to EC2 via SSH') {
            when {
                anyOf {
                    expression { env.FRONTEND_CHANGED == 'true' }
                    expression { env.BACKEND_CHANGED == 'true' }
                }
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        echo "🚀 SSH into EC2 and deploy"

                        ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" ${DEPLOY_USER}@${DEPLOY_HOST} '
                            set -e
                            mkdir -p ${DEPLOY_PATH}
                            cd ${DEPLOY_PATH}

                            echo "📥 Cloning/Updating docker-compose repo..."
                            if [ ! -d ".git" ]; then
                                git clone https://github.com/UdayKiranChilumula/Docker-UseCase.git . 
                            else
                                git pull origin main
                            fi

                            echo "📦 Pulling latest Docker images..."
                            TAG=${BUILD_NUMBER} docker-compose pull

                            echo "🔧 Starting containers..."
                            TAG=${BUILD_NUMBER} docker-compose up -d

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
