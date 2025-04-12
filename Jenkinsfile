pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'nextjs-app'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        CONTAINER_NAME = 'nextjs-app-container'
        DOCKER_REGISTRY = 'your-registry-url' // Update with your Docker registry URL
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
      
        stage('Build Docker Image') {
        steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest"
            }
        }
        
        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_CREDENTIALS')]) {
                    sh "echo ${DOCKER_CREDENTIALS} | docker login ${DOCKER_REGISTRY} -u username --password-stdin"
                    sh "docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                // Stop and remove existing container
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"
                
                // Run new container
                sh "docker run -d -p 3000:3000 --name ${CONTAINER_NAME} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }
    }
    
    post {
        always {
            // Clean up Docker images
            sh "docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true"
            sh "docker rmi ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG} || true"
            sh "docker rmi ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest || true"
        }
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'CI/CD pipeline failed!'
        }
    }
}
