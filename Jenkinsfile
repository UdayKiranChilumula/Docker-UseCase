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

        // Generic Webhook passes repo name as 'REPO_NAME'
        REPO_NAME = "${env.REPO_NAME}"
    }

    stages {
        stage('Clone Repos') {
            steps {
                script {
                    if (REPO_NAME == 'Frontend') {
                        dir('frontend') {
                            git url: "${FRONTEND_REPO}", branch: 'main'
                        }
                    } else if (REPO_NAME == 'Backend') {
                        dir('backend') {
                            git url: "${BACKEND_REPO}", branch: 'main'
                        }
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (REPO_NAME == 'Frontend') {
                        dir('frontend') {
                            sh """
                                docker build -t ${FRONTEND_IMAGE} .
                                docker tag ${FRONTEND_IMAGE} udaykiranchilumula/${FRONTEND_IMAGE}
                            """
                        }
                    } else if (REPO_NAME == 'Backend') {
                        dir('backend') {
                            sh """
                                docker build -t ${BACKEND_IMAGE} .
                                docker tag ${BACKEND_IMAGE} udaykiranchilumula/${BACKEND_IMAGE}
                            """
                        }
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    if (REPO_NAME == 'Frontend' || REPO_NAME == 'Backend') {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh """
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            """
                            if (REPO_NAME == 'Frontend') {
                                sh "docker push udaykiranchilumula/${FRONTEND_IMAGE}"
                            } else if (REPO_NAME == 'Backend') {
                                sh "docker push udaykiranchilumula/${BACKEND_IMAGE}"
                            }
                        }
                    }
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
                        mkdir -p ${DEPLOY_PATH}
                        cd ${DEPLOY_PATH}

                        echo "📥 Cloning/Updating docker-compose repo..."
                        if [ ! -d ".git" ]; then
                            git clone https://github.com/UdayKiranChilumula/Docker-UseCase.git . 
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
            echo '🧹 Cleaning up...'
            dir('frontend') {
                deleteDir()
            }
            dir('backend') {
                deleteDir()
            }
        }
    }
}
