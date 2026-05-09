pipeline {
    agent any
    
    environment {
        APP_NAME = "color-checker-service"
        RELEASE = "1.0.0"
        DOCKER_USER = "oredollar"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {
        stage("Cleanup workspace") {
            steps {
                cleanWs()
            }
        }
        
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/oredollars/color-checker-service.git'
            }
        }
        
        stage("Build Docker Image") {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }

stage('Trigger Manifest Update') {
    steps {
        script {
            echo 'triggering kubernetes-deployment job'

            build job: 'color-checker-k8s-update',
            parameters: [
                string(name: 'DOCKER_TAG', value: "${DOCKER_TAG}")
            ]
        }
    }
}
            ]
        }
    }
}

    }
    
    post {
        success {
            echo "Successfully built and pushed ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Build or push failed!"
        }
    }
}
