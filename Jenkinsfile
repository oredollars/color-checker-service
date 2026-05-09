pipeline {
    agent any

    environment {
        IMAGE_NAME = "oredollar/color-checker-service"
        DOCKER_TAG = "1.0.0-${BUILD_NUMBER}"
    }

    stages {

        stage('Cleanup workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
            steps {
                git branch: 'main',
                credentialsId: 'github',
                url: 'https://github.com/oredollars/color-checker-service.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {

                    sh """
                    docker build -t ${IMAGE_NAME}:${DOCKER_TAG} .
                    """

                    sh """
                    docker tag ${IMAGE_NAME}:${DOCKER_TAG} ${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub',
                            passwordVariable: 'PASS',
                            usernameVariable: 'USER'
                        )
                    ]) {

                        sh """
                        echo \$PASS | docker login -u \$USER --password-stdin

                        docker push ${IMAGE_NAME}:${DOCKER_TAG}

                        docker push ${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }

        stage('Trigger Manifest Update') {
            steps {
                script {

                    echo 'triggering kubernetes-deployment job'

                    build(
                        job: 'color-checker-k8s-update',
                        parameters: [
                            string(
                                name: 'DOCKER_TAG',
                                value: "${DOCKER_TAG}"
                            )
                        ]
                    )
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }

        failure {
            echo 'Build or push failed!'
        }
    }
}
