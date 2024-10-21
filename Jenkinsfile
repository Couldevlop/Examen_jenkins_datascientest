pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub_credentials'
        DOCKER_IMAGE_MOVIE_SERVICE = 'thomcoul/movie_service'
        DOCKER_IMAGE_CAST_SERVICE = 'thomcoul/cast_service'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/DataScientest/Jenkins_devops_exams.git'
            }
        }

        stage('Build and Push Docker Images') {
            steps {
                script {
                    // Utiliser docker.build pour créer toutes les images
                    def dockerImageMovie = docker.build("${DOCKER_IMAGE_MOVIE_SERVICE}:${env.BRANCH_NAME}", "movie-service")
                    def dockerImageCast = docker.build("${DOCKER_IMAGE_CAST_SERVICE}:${env.BRANCH_NAME}", "cast-service")
                    
                    // Pusher toutes les images vers Docker Hub
                    docker.withRegistry('', "${DOCKER_CREDENTIALS_ID}") {
                        dockerImageMovie.push()
                        dockerImageCast.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'dev') {
                        sh 'helm upgrade --install movie-service helm-charts/movie-service --namespace dev --values helm-charts/movie-service/values-dev.yaml'
                        sh 'helm upgrade --install cast-service helm-charts/cast-service --namespace dev --values helm-charts/cast-service/values-dev.yaml'
                        sh 'helm upgrade --install nginx helm-charts/nginx --namespace dev --values helm-charts/nginx/values-dev.yaml'
                    } else if (env.BRANCH_NAME == 'qa') {
                        sh 'helm upgrade --install movie-service helm-charts/movie-service --namespace qa --values helm-charts/movie-service/values-qa.yaml'
                        sh 'helm upgrade --install cast-service helm-charts/cast-service --namespace qa --values helm-charts/cast-service/values-qa.yaml'
                        sh 'helm upgrade --install nginx helm-charts/nginx --namespace qa --values helm-charts/nginx/values-qa.yaml'
                    } else if (env.BRANCH_NAME == 'staging') {
                        sh 'helm upgrade --install movie-service helm-charts/movie-service --namespace staging --values helm-charts/movie-service/values-staging.yaml'
                        sh 'helm upgrade --install cast-service helm-charts/cast-service --namespace staging --values helm-charts/cast-service/values-staging.yaml'
                        sh 'helm upgrade --install nginx helm-charts/nginx --namespace staging --values helm-charts/nginx/values-staging.yaml'
                    } else if (env.BRANCH_NAME == 'master') {
                        sh 'helm upgrade --install movie-service helm-charts/movie-service --namespace prod --values helm-charts/movie-service/values-prod.yaml'
                        sh 'helm upgrade --install cast-service helm-charts/cast-service --namespace prod --values helm-charts/cast-service/values-prod.yaml'
                        sh 'helm upgrade --install nginx helm-charts/nginx --namespace prod --values helm-charts/nginx/values-prod.yaml'
                    }
                }
            }
        }
    }
}
