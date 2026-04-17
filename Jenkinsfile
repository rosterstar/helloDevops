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
                // Рекомендуется также использовать withCredentials здесь для безопасности
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
                    withCredentials([usernamePassword(
                        credentialsId: 'harbor-deploy',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no pod2user@192.168.65.5 "
                            echo '${PASS}' | docker login ${REGISTRY} -u ${USER} --password-stdin &&
                            docker pull ${REGISTRY}/${IMAGE}:${BUILD_NUMBER} &&
                            docker stop hello || true &&
                            docker rm hello || true &&
                            docker run -d -p 8080:8080 --name hello ${REGISTRY}/${IMAGE}:${BUILD_NUMBER}
                        "
                        """
                    }
                }
            }
        }
    }
}


