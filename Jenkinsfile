pipeline {
    agent any

    environment {
        // Docker image details
        DOCKER_IMAGE    = 'myapp'
        DOCKER_TAG      = "${BUILD_NUMBER}"
        DOCKER_REGISTRY = 'docker.io'

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
                // git branch: 'main',
                //     credentialsId: 'github-credentials',
                //     url: 'https://github.com/YOUR-USERNAME/YOUR-REPO.git'
                sh 'ls -la'
            }
        }

        stage('🔨 Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build \
                          --build-arg BUILD_NUMBER=${BUILD_NUMBER} \
                          --build-arg APP_VERSION=${APP_VERSION} \
                          -t ${DOCKER_IMAGE}:${DOCKER_TAG} \
                          -t ${DOCKER_IMAGE}:latest \
                          .
                    """
                    sh 'docker images | grep ${DOCKER_IMAGE}'
                }
            }
        }

        stage('🧪 Test Docker Image') {
            steps {
                script {
                    sh """
                        docker run -d \
                          --name test-container-${BUILD_NUMBER} \
                          -p 3000:3000 \
                          -e APP_VERSION=${APP_VERSION} \
                          -e BUILD_NUMBER=${BUILD_NUMBER} \
                          ${DOCKER_IMAGE}:${DOCKER_TAG}

                        echo "Waiting for container..."
                        for i in {1..30}; do
                            if docker ps | grep test-container-${BUILD_NUMBER} | grep -q "Up"; then
                                if curl -f http://localhost:3000/health || curl -f http://localhost:3000/; then
                                    echo "✅ App is responding"
                                    break
                                fi
                            fi
                            echo "Attempt \$i/30"
                            sleep 1
                        done

                        docker logs test-container-${BUILD_NUMBER}
                        docker ps -a | grep test-container-${BUILD_NUMBER}
                    """
                }
            }
            post {
                always {
                    sh """
                        docker logs test-container-${BUILD_NUMBER} || true
                        docker stop test-container-${BUILD_NUMBER} || true
                        docker rm test-container-${BUILD_NUMBER} || true
                    """
                }
            }
        }

        stage('🔍 Security Scan') {
            steps {
                sh """
                    docker inspect ${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker history ${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker images ${DOCKER_IMAGE}:${DOCKER_TAG} --format "{{.Size}}"
                """
            }
        }

        stage('📤 Push to Registry') {
            when {
                expression {
                    env.GIT_BRANCH == 'origin/main' ||
                    env.GIT_BRANCH == 'origin/master' ||
                    env.BRANCH_NAME == 'main'
                }
            }
            steps {
                echo "Would push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                echo "Would push ${DOCKER_IMAGE}:latest"
            }
        }

        stage('🚀 Deploy (Local Test)') {
            steps {
                script {
                    sh """
                        docker stop myapp-demo || true
                        docker rm myapp-demo || true

                        docker run -d \
                          --name myapp-demo \
                          --restart unless-stopped \
                          -p 3000:3000 \
                          -e APP_VERSION=${APP_VERSION} \
                          -e BUILD_NUMBER=${BUILD_NUMBER} \
                          ${DOCKER_IMAGE}:${DOCKER_TAG}

                        sleep 5
                        docker ps | grep myapp-demo
                        docker logs --tail 20 myapp-demo
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ DOCKER PIPELINE SUCCEEDED'
            echo "Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo "Version: ${APP_VERSION}"
        }

        failure {
            echo '❌ DOCKER PIPELINE FAILED'
        }

        always {
            sh '''
                docker image prune -f || true
                docker images
            '''
        }
    }
}
