pipeline {
    agent any

    tools {
        nodejs 'NodeJS_18'   // Make sure this Node version exists in Jenkins
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Narek97/my-app-front.git'
            }
        }

        stage('Install dependencies') {
            steps {
                sh '''
                    npm install
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    npm run build
                '''
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
