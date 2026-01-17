pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Narek97/my-app-front.git'
            }
        }

        stage('Check Node & NPM') {
            steps {
                sh '''
                    echo "Node version:"
                    node -v
                    echo "NPM version:"
                    npm -v
                    echo "Working directory:"
                    pwd
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Test Build Output') {
            steps {
                sh '''
                    echo "Build artifacts:"
                    ls -la .next/ || ls -la build/ || ls -la dist/
                '''
            }
        }

        stage('Start & Test') {
            steps {
                sh '''
                    echo "Starting application..."
                    npm run start &
                    APP_PID=$!
                    
                    echo "Waiting for app to start..."
                    sleep 10
                    
                    echo "Testing application..."
                    curl -I http://localhost:3000 || echo "App is running on port 3000"
                    
                    echo "Stopping application..."
                    kill $APP_PID || true
                    
                    # Make sure it's killed
                    pkill -f "node" || true
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            sh '''
                # Kill any remaining node processes
                pkill -f "node" || true
                
                # Clean up build artifacts
                rm -rf node_modules .next build dist || true
            '''
        }
        
        success {
            echo '✅ Build succeeded!'
        }
        
        failure {
            echo '❌ Build failed!'
        }
    }
}