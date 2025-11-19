pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "my-web-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Test Image') {
            steps {
                echo 'Testing Docker image...'
                script {
                    sh "docker images | grep ${DOCKER_IMAGE}"
                }
            }
        }
        
        stage('Run Container') {
    steps {
        echo 'Running container for testing...'
        script {
            sh '''
                # Remove existing container if present
                docker rm -f test-container 2>/dev/null || true
                
                # Start container
                docker run -d --name test-container -p 8081:80 ${DOCKER_IMAGE}:${DOCKER_TAG}

                echo "Waiting for container to become ready..."

                # Retry up to 30 seconds
                for i in {1..30}; do
                    if curl -fs http://localhost:8081 > /dev/null; then
                        echo "Container is responding!"
                        break
                    fi
                    echo "Still waiting... ($i/30)"
                    sleep 1
                done

                # Final check
                if ! curl -fs http://localhost:8081 > /dev/null; then
                    echo "Container failed to start after waiting."
                    exit 1
                fi

                echo "Container is running successfully!"
            '''
        }
    }
}
        
        stage('Cleanup') {
            steps {
                echo 'Cleaning up test container...'
                script {
                    sh 'docker stop test-container || true'
                    sh 'docker rm test-container || true'
                }
            }
        }
    }
    
    post {
        success {
            echo "✅ Pipeline completed! Docker image ${DOCKER_IMAGE}:${DOCKER_TAG} is ready!"
        }
        failure {
            echo '❌ Pipeline failed!'
            script {
                sh 'docker stop test-container 2>/dev/null || true'
                sh 'docker rm test-container 2>/dev/null || true'
            }
        }
    }
}