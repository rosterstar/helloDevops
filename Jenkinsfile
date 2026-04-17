pipeline {
    agent any

    environment {
        REGISTRY = "registry.local"
        IMAGE = "devops/hello-devops"
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE:$BUILD_NUMBER .'
            }
        }

        stage('Login') {
            steps {
                sh 'docker logout $REGISTRY || true'
                withCredentials([usernamePassword(
                    credentialsId: 'harbor-jenkins',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    // Используем одинарные кавычки для защиты спецсимволов в логине робота
                    sh "echo '\$PASS' | docker login $REGISTRY -u '\$USER' --password-stdin"
                }
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
                        usernameVariable: 'D_USER',
                        passwordVariable: 'D_PASS'
                    )]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no pod2user@192.168.65.5 "
                            echo '${D_PASS}' | docker login ${REGISTRY} -u '${D_USER}' --password-stdin &&
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
