pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKERHUB_USERNAME = 'your-dockerhub-username'
        EC2_HOST = 'your-ec2-public-ip'
        EC2_USER = 'ubuntu'
        SSH_KEY = credentials('ec2-ssh-key')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sakshi-nikhade-07/devops-project'
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('backend') {
                    sh """
                        docker build -t ${DOCKERHUB_USERNAME}/mean-backend:${BUILD_NUMBER} .
                        docker tag ${DOCKERHUB_USERNAME}/mean-backend:${BUILD_NUMBER} \
                                   ${DOCKERHUB_USERNAME}/mean-backend:latest
                    """
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    sh """
                        docker build -t ${DOCKERHUB_USERNAME}/mean-frontend:${BUILD_NUMBER} .
                        docker tag ${DOCKERHUB_USERNAME}/mean-frontend:${BUILD_NUMBER} \
                                   ${DOCKERHUB_USERNAME}/mean-frontend:latest
                    """
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh """
                    echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                    docker push ${DOCKERHUB_USERNAME}/mean-backend:${BUILD_NUMBER}
                    docker push ${DOCKERHUB_USERNAME}/mean-backend:latest
                    docker push ${DOCKERHUB_USERNAME}/mean-frontend:${BUILD_NUMBER}
                    docker push ${DOCKERHUB_USERNAME}/mean-frontend:latest
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh """
                        scp -o StrictHostKeyChecking=no docker-compose.yml ${EC2_USER}@${EC2_HOST}:/home/${EC2_USER}/

                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                            docker pull ${DOCKERHUB_USERNAME}/mean-backend:latest
                            docker pull ${DOCKERHUB_USERNAME}/mean-frontend:latest
                            docker-compose down
                            docker-compose up -d
                            docker system prune -f
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed! Check logs.'
        }
        always {
            sh 'docker logout'
        }
    }
}
