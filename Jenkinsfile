pipeline {
    agent any
    
    environment {
        // Docker image details
        DOCKER_IMAGE = 'myapp'
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKER_REGISTRY = 'docker.io'  // Change to your registry
        
        // Application version
        APP_VERSION = "1.0.${BUILD_NUMBER}"
    }
    
    stages {
        stage('🔍 Environment Check') {
            steps {
                echo '=== Checking Docker Environment ==='
                sh '''
                    docker --version
                    docker info
                    echo "Build Number: ${BUILD_NUMBER}"
                    echo "Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                '''
            }
        }
        
        stage('📋 Checkout') {
            steps {
                echo 'Checking out source code...'
                // If using Git, uncomment:
                // git branch: 'main',
                //     credentialsId: 'github-credentials',
                //     url: 'https://github.com/YOUR-USERNAME/YOUR-REPO.git'
                
                // For now, we'll assume code is in workspace
                sh 'ls -la'
            }
        }
        
        stage('🔨 Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    // Build image with build number as tag
                    sh """
                        docker build \
                          --build-arg BUILD_NUMBER=${BUILD_NUMBER} \
                          --build-arg APP_VERSION=${APP_VERSION} \
                          -t ${DOCKER_IMAGE}:${DOCKER_TAG} \
                          -t ${DOCKER_IMAGE}:latest \
                          .
                    """
                    
                    // List images
                    sh 'docker images | grep ${DOCKER_IMAGE}'
                }
            }
        }
        
        stage('🧪 Test Docker Image') {
            steps {
                echo 'Testing Docker image...'
                script {
                    // Run container for testing
                    sh """
                        # Start container
                        docker run -d \
                          --name test-container-${BUILD_NUMBER} \
                          -p 3001:3000 \
                          -e APP_VERSION=${APP_VERSION} \
                          -e BUILD_NUMBER=${BUILD_NUMBER} \
                          ${DOCKER_IMAGE}:${DOCKER_TAG}
                        
                        # Wait for container to start
                        sleep 5
                        
                        # Test health endpoint
                        curl -f http://localhost:3001/health || exit 1
                        
                        # Test main endpoint
                        curl -f http://localhost:3001/ || exit 1
                        
                        echo "✅ Container tests passed!"
                    """
                }
            }
            post {
                always {
                    // Clean up test container
                    sh """
                        docker stop test-container-${BUILD_NUMBER} || true
                        docker rm test-container-${BUILD_NUMBER} || true
                    """
                }
            }
        }
        
        stage('🔍 Security Scan') {
            steps {
                echo 'Scanning image for vulnerabilities...'
                script {
                    // Basic image inspection
                    sh """
                        echo "=== Image Details ==="
                        docker inspect ${DOCKER_IMAGE}:${DOCKER_TAG}
                        
                        echo "=== Image History ==="
                        docker history ${DOCKER_IMAGE}:${DOCKER_TAG}
                        
                        echo "=== Image Size ==="
                        docker images ${DOCKER_IMAGE}:${DOCKER_TAG} --format "{{.Size}}"
                    """
                    
                    // In production, you would use tools like:
                    // - Trivy: docker run aquasec/trivy image ${DOCKER_IMAGE}:${DOCKER_TAG}
                    // - Snyk: snyk container test ${DOCKER_IMAGE}:${DOCKER_TAG}
                }
            }
        }
        
        stage('📤 Push to Registry') {
            when {
                expression { 
                    // Only push on master/main branch
                    return env.GIT_BRANCH == 'origin/main' || env.GIT_BRANCH == 'origin/master' || env.BRANCH_NAME == 'main'
                }
            }
            steps {
                echo 'Pushing image to Docker registry...'
                script {
                    // In production, use withCredentials to login to registry
                    // For now, we'll skip the actual push
                    echo "Would push: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    echo "Would push: ${DOCKER_IMAGE}:latest"
                    
                    // Uncomment when ready to push:
                    /*
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                            docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${DOCKER_IMAGE}:latest
                            docker logout
                        '''
                    }
                    */
                }
            }
        }
        
        stage('🚀 Deploy (Local Test)') {
            steps {
                echo 'Deploying container locally...'
                script {
                    sh """
                        # Stop and remove old container if exists
                        docker stop myapp-demo || true
                        docker rm myapp-demo || true
                        
                        # Run new container
                        docker run -d \
                          --name myapp-demo \
                          --restart unless-stopped \
                          -p 3000:3000 \
                          -e APP_VERSION=${APP_VERSION} \
                          -e BUILD_NUMBER=${BUILD_NUMBER} \
                          ${DOCKER_IMAGE}:${DOCKER_TAG}
                        
                        # Verify deployment
                        sleep 3
                        docker ps | grep myapp-demo
                        curl -f http://localhost:3000/health
                        
                        echo "✅ Deployment successful!"
                        echo "🌐 Access at: http://localhost:3000"
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo '======================================'
            echo '✅ DOCKER PIPELINE SUCCEEDED!'
            echo "Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo "Version: ${APP_VERSION}"
            echo "Access app at: http://localhost:3000"
            echo '======================================'
        }
        
        failure {
            echo '======================================'
            echo '❌ DOCKER PIPELINE FAILED!'
            echo "Check logs for details"
            echo '======================================'
        }
        
        always {
            echo 'Cleaning up...'
            sh '''
                # Remove dangling images
                docker image prune -f || true
                
                # Show current images
                echo "=== Current Docker Images ==="
                docker images
            '''
        }
    }
}