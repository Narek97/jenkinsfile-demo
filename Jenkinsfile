stage('📤 Push to Docker Hub') {
    steps {
        script {
            withCredentials([usernamePassword(
                credentialsId: 'dockerhub-credentials',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )]) {
                sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    
                    # Tag image with Docker Hub username
                    docker tag myapp:${BUILD_NUMBER} $DOCKER_USER/myapp:${BUILD_NUMBER}
                    docker tag myapp:latest $DOCKER_USER/myapp:latest
                    
                    # Push to Docker Hub
                    docker push $DOCKER_USER/myapp:${BUILD_NUMBER}
                    docker push $DOCKER_USER/myapp:latest
                    
                    docker logout
                '''
            }
        }
    }
}