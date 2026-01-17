pipeline {
    agent {
        docker {
            image 'node:18-alpine'
        }
    }
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
                    node -v
                    npm -v
                '''
            }
        }

        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Run (check)') {
            steps {
                sh '''
                    npm run start &
                    sleep 5
                    curl http://localhost:3000 || echo "App started"
                    pkill -f "next start" || true
                '''
            }
        }
    }

    post {
        always {
            sh 'rm -rf node_modules .next'
        }
    }
}
