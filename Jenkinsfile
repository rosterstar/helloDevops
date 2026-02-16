pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/rosterstar/hello-devops.git'
            }
        }

        stage('Build Docker image') {
            steps {
                sh 'docker build -t hello-devops:latest .'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                  docker stop hello || true
                  docker rm hello || true
                  docker run -d \
                    --name hello \
                    -p 8080:8080 \
                    hello-devops:latest
                '''
            }
        }
    }
}

