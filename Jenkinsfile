pipeline {
    agent any

    environment {
        REGISTRY = "registry.local"
        IMAGE = "hello-devops"
    }

    stages {

        stage('Build') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE:$BUILD_NUMBER .'
            }
        }

        stage('Login') {
            steps {
                sh 'echo "PASSWORD" | docker login $REGISTRY -u admin --password-stdin'
            }
        }

        stage('Push') {
            steps {
                sh 'docker push $REGISTRY/$IMAGE:$BUILD_NUMBER'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                ssh user@APP_VM "
                    docker login registry.local -u admin -p 123456 &&
                    docker pull $REGISTRY/$IMAGE:$BUILD_NUMBER &&
                    docker stop hello || true &&
                    docker rm hello || true &&
                    docker run -d -p 8080:8080 \
                        --name hello \
                        $REGISTRY/$IMAGE:$BUILD_NUMBER
                "
                '''
            }
        }
    }
}
