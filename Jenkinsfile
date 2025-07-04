pipeline {
    agent any

    environment {
        FRONTEND_REPO = 'https://github.com/UdayKiranChilumula/Frontend.git'
        BACKEND_REPO  = 'https://github.com/UdayKiranChilumula/Backend.git'

        FRONTEND_IMAGE = 'estate-frontend'
        BACKEND_IMAGE  = 'estate-backend'

        AWS_ACCOUNT_ID = '440744254638'
        REGION = 'us-east-1'
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com"

        DEPLOY_USER = 'ec2-user'
        DEPLOY_HOST = '13.218.63.126'
        DEPLOY_PATH = '/home/ec2-user/estate-deployment'

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
                            sh "docker build -t ${FRONTEND_IMAGE} ."
                        }
                    } else if (REPO_NAME == 'Backend') {
                        dir('backend') {
                            sh "docker build -t ${BACKEND_IMAGE} ."
                        }
                    }
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                        sh """
                            aws ecr get-login-password --region ${REGION} | \
                            docker login --username AWS --password-stdin ${ECR_URI}
                        """

                        if (REPO_NAME == 'Frontend') {
                            sh """
                                docker tag ${FRONTEND_IMAGE} ${ECR_URI}/${FRONTEND_IMAGE}
                                docker push ${ECR_URI}/${FRONTEND_IMAGE}
                            """
                        } else if (REPO_NAME == 'Backend') {
                            sh """
                                docker tag ${BACKEND_IMAGE} ${ECR_URI}/${BACKEND_IMAGE}
                                docker push ${ECR_URI}/${BACKEND_IMAGE}
                            """
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

                        echo "🔐 Logging into Amazon ECR using EC2 IAM Role"
                        aws ecr get-login-password --region ${REGION} | \
                        docker login --username AWS --password-stdin ${ECR_URI}

                        echo "📦 Pulling latest Docker images..."
                        docker-compose pull

                        echo "🚀 Starting containers..."
                        docker-compose up -d --pull always

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
