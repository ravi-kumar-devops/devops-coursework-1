pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDS = credentials('dockerhub')
        APP_VERSION = "${env.BUILD_NUMBER}"
        DOCKER_IMAGE = "your-actual-dockerhub-username/cw2-server"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        PROD_SERVER = "actual-production-server-ip"
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
                sh "echo ${DOCKER_HUB_CREDS_PSW} | docker login -u ${DOCKER_HUB_CREDS_USR} --password-stdin"
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
