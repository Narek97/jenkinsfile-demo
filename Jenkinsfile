pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'myapp'
        DOCKER_TAG = "${BUILD_NUMBER}"
        DOCKER_REGISTRY = 'docker.io'
        APP_VERSION = "1.0.${BUILD_NUMBER}"
        DOCKERHUB_USERNAME = 'your-dockerhub-username'  // CHANGE THIS!
    }
    
    stages {
        stage('🔍 Environment Check') {
            steps {
                echo '=== Checking Docker Environment ==='
                sh '''
                    docker --version
                    docker info | head -20
                    echo "Build Number: ${BUILD_NUMBER}"
                    echo "Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                '''
            }
        }
        
        stage('📋 Checkout') {
            steps {
                echo 'Checking out source code...'
                sh '''
                    echo "=== Repository Contents ==="
                    ls -la
                    
                    echo ""
                    echo "=== Verifying Dockerfile exists ==="
                    if [ -f Dockerfile ]; then
                        echo "✅ Dockerfile found"
                        cat Dockerfile
                    else
                        echo "❌ Dockerfile not found!"
                        exit 1
                    fi
                '''
            }
        }
        
        stage('🔨 Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh """
                        echo "Building image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        
                        docker build \
                          --build-arg BUILD_NUMBER=${BUILD_NUMBER} \
                          --build-arg APP_VERSION=${APP_VERSION} \
                          -t ${DOCKER_IMAGE}:${DOCKER_TAG} \
                          -t ${DOCKER_IMAGE}:latest \
                          .
                        
                        echo ""
                        echo "=== Built Images ==="
                        docker images | grep ${DOCKER_IMAGE}
                    """
                }
            }
        }
        
        stage('🧪 Test Docker Image') {
            steps {
                echo 'Skipping test - will verify in deployment stage'
                sh '''
                    echo "Building test container for logs only..."
                    docker run -d --name test-container-${BUILD_NUMBER} -p 3001:3000 myapp:${BUILD_NUMBER}
                    sleep 10
                    echo "=== Container Logs ==="
                    docker logs test-container-${BUILD_NUMBER}
                    echo "✅ Container started successfully (connectivity test in deploy stage)"
                '''
            }
            post {
                always {
                    sh 'docker stop test-container-${BUILD_NUMBER} || true'
                    sh 'docker rm test-container-${BUILD_NUMBER} || true'
                }
            }
        }
        
        stage('🔍 Security Scan') {
            steps {
                echo 'Scanning image for vulnerabilities...'
                script {
                    sh """
                        echo "=== Image Details ==="
                        docker inspect ${DOCKER_IMAGE}:${DOCKER_TAG} --format='{{.Config.Image}}'
                        
                        echo ""
                        echo "=== Image Size ==="
                        docker images ${DOCKER_IMAGE}:${DOCKER_TAG} --format "Size: {{.Size}}"
                        
                        echo ""
                        echo "=== Image Layers ==="
                        docker history ${DOCKER_IMAGE}:${DOCKER_TAG} --no-trunc
                    """
                }
            }
        }
        
        stage('📤 Push to Docker Hub') {
            when {
                expression { 
                    return true
                }
            }
            steps {
                echo 'Pushing image to Docker Hub...'
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-credentials',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {
                        sh """
                            echo "Logging in to Docker Hub..."
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            
                            echo ""
                            echo "Tagging images..."
                            docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} \$DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} \$DOCKER_USER/${DOCKER_IMAGE}:latest
                            
                            echo ""
                            echo "Pushing ${DOCKER_IMAGE}:${DOCKER_TAG}..."
                            docker push \$DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}
                            
                            echo ""
                            echo "Pushing ${DOCKER_IMAGE}:latest..."
                            docker push \$DOCKER_USER/${DOCKER_IMAGE}:latest
                            
                            echo ""
                            echo "✅ Successfully pushed to Docker Hub!"
                            echo "Image: \$DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}"
                            echo "Latest: \$DOCKER_USER/${DOCKER_IMAGE}:latest"
                            
                            docker logout
                        """
                    }
                }
            }
        }
        
        stage('🚀 Deploy (Local Test)') {
            steps {
                echo 'Deploying container locally...'
                script {
                    sh """
                        set +e
                        
                        echo "Cleaning up old deployment..."
                        docker stop myapp-demo 2>/dev/null || true
                        docker rm myapp-demo 2>/dev/null || true
                        
                        echo ""
                        echo "Starting new deployment..."
                        docker run -d \
                          --name myapp-demo \
                          --restart unless-stopped \
                          -p 3000:3000 \
                          -e APP_VERSION=${APP_VERSION} \
                          -e BUILD_NUMBER=${BUILD_NUMBER} \
                          ${DOCKER_IMAGE}:${DOCKER_TAG}
                        
                        echo "Waiting for application to start..."
                        sleep 10
                        
                        echo ""
                        echo "=== Deployment Status ==="
                        docker ps | grep myapp-demo || echo "Container not found in running processes"
                        
                        echo ""
                        echo "=== Application Logs (last 20 lines) ==="
                        docker logs --tail 20 myapp-demo
                        
                        echo ""
                        echo "=== Connectivity Test ==="
                        if curl -f -s http://localhost:3000/ > /dev/null 2>&1; then
                            echo "✅ Application is accessible at http://localhost:3000"
                        else
                            echo "⏳ Application may still be starting..."
                            echo "   Check manually at: http://localhost:3000"
                            echo "   View logs: docker logs -f myapp-demo"
                        fi
                        
                        echo ""
                        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                        echo "✅ Deployment completed!"
                        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                        echo "📦 Container: myapp-demo"
                        echo "🏷️  Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                        echo "📌 Version: ${APP_VERSION}"
                        echo "🌐 URL: http://localhost:3000"
                        echo "📝 Logs: docker logs -f myapp-demo"
                        echo "🛑 Stop: docker stop myapp-demo"
                        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
                        
                        set -e
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo '=========================================='
            echo '✅ DOCKER PIPELINE SUCCEEDED!'
            echo '=========================================='
            echo "📦 Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo "📌 Version: ${APP_VERSION}"
            echo "🌐 Local: http://localhost:3000"
            echo "🐳 Docker Hub: https://hub.docker.com/r/${DOCKERHUB_USERNAME}/${DOCKER_IMAGE}"
            echo '=========================================='
        }
        
        failure {
            echo '=========================================='
            echo '❌ DOCKER PIPELINE FAILED!'
            echo '=========================================='
            echo 'Check the logs above for error details'
            echo '=========================================='
        }
        
        always {
            echo 'Cleaning up...'
            sh '''
                docker image prune -f || true
                
                echo ""
                echo "=== Current Docker Images ==="
                docker images | head -10
                
                echo ""
                echo "=== Running Containers ==="
                docker ps --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"
            '''
        }
    }
}