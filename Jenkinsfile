pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'robertsecon'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    sh "docker build -t ${DOCKERHUB_USER}/movie-service:${IMAGE_TAG} ./movie-service"
                    sh "docker build -t ${DOCKERHUB_USER}/cast-service:${IMAGE_TAG} ./cast-service"
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        sh "docker push ${DOCKERHUB_USER}/movie-service:${IMAGE_TAG}"
                        sh "docker push ${DOCKERHUB_USER}/cast-service:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                script {
                    sh """
                        helm upgrade --install microservices-dev ./helm/microservices \
                            -f ./helm/microservices/values-dev.yaml \
                            -n dev \
                            --set movieService.tag=${IMAGE_TAG} \
                            --set castService.tag=${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                script {
                    sh """
                        helm upgrade --install microservices-qa ./helm/microservices \
                            -f ./helm/microservices/values-qa.yaml \
                            -n qa \
                            --set movieService.tag=${IMAGE_TAG} \
                            --set castService.tag=${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    sh """
                        helm upgrade --install microservices-staging ./helm/microservices \
                            -f ./helm/microservices/values-staging.yaml \
                            -n staging \
                            --set movieService.tag=${IMAGE_TAG} \
                            --set castService.tag=${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Deploy to Production?', ok: 'Deploy'
                script {
                    sh """
                        helm upgrade --install microservices-prod ./helm/microservices \
                            -f ./helm/microservices/values-prod.yaml \
                            -n prod \
                            --set movieService.tag=${IMAGE_TAG} \
                            --set castService.tag=${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        always {
            sh "docker logout"
        }
    }
}