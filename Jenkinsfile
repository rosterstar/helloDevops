pipeline {
    agent any

    environment {
        REGISTRY = "registry.local"
        PROJECT = "devops"
        IMAGE = "hello-devops"
        FULL_IMAGE_PATH = "${REGISTRY}/${PROJECT}"
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t ${FULL_IMAGE_PATH}:${BUILD_NUMBER} .'
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
                    sh 'echo "$PASS" | docker login $REGISTRY -u "$USER" --password-stdin'
                }
            }
        }

        stage('Push') {
            steps {
                sh 'docker push ${FULL_IMAGE_PATH}'
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
