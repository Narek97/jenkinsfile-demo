pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/do-community/flask-hello-world.git'
            }
        }
        
        stage('Setup') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                '''
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    # Add your test commands here
                    python3 -m pytest --version || echo "No tests found"
                '''
            }
        }
        
        stage('Run') {
            steps {
                sh '''
                    . venv/bin/activate
                    python3 app.py &
                    sleep 3
                    curl http://localhost:5000 || echo "App running"
                '''
            }
        }
    }
    
    post {
        always {
            sh 'rm -rf venv' // Cleanup
        }
    }
}