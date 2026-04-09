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
                sh 'echo "123456" | docker login $REGISTRY -u admin --password-stdin'
            }
        }

        stage('Push') {
            steps {
                sh 'docker push $REGISTRY/$IMAGE:$BUILD_NUMBER'
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['app-server-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no pod2user@192.168.65.5 "
                            docker login ${REGISTRY} -u admin -p 123456 &&
                            docker pull ${REGISTRY}/${IMAGE}:${BUILD_NUMBER} &&
                            docker stop hello || true &&
                            docker rm hello || true &&
                            docker run -d -p 8080:8080 --name hello ${REGISTRY}/${IMAGE}:${BUILD_NUMBER}
                        "
                    """
                } // <--- Вот эта скобка для sshagent была потеряна
            }
        }
    }
}
