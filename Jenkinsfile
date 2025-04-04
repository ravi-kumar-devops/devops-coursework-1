pipeline {
    agent any
    
    environment {
        APP_VERSION = "${env.BUILD_NUMBER}"
        DOCKER_IMAGE = "ravi787/cw2-server"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        PROD_SERVER = "192.168.1.100"  // Replace with your actual production server IP
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
            }
        }
        
        stage('Test Container') {
            steps {
                sh "docker run -d --name test-${BUILD_NUMBER} ${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh "docker ps | grep test-${BUILD_NUMBER}"
                sh "docker exec test-${BUILD_NUMBER} node -v || exit 1"
                sh "docker stop test-${BUILD_NUMBER}"
                sh "docker rm test-${BUILD_NUMBER}"
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                // Using direct credentials (less secure but for troubleshooting)
                sh '''
                echo "3AQ9uAEpffeH" | docker login -u ravi787 --password-stdin
                '''
                sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh "docker push ${DOCKER_IMAGE}:latest"
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                sshagent(['prod-server-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${PROD_SERVER} '
                        kubectl set image deployment/cw2-server cw2-server=${DOCKER_IMAGE}:${DOCKER_TAG}
                        kubectl rollout status deployment/cw2-server
                        '
                    """
                }
            }
        }
    }
    
    post {
        always {
            sh "docker logout"
        }
    }
}