pipeline {
    agent any

    stages {
        stage('Authenticate with Docker Hub') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub-account', url: 'https://index.docker.io/v1/') {
                    echo 'Logged in to Docker Hub'
                }
            }
        }
        
        stage('Push images to Docker Hub') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub-account', url: 'https://index.docker.io/v1/') {
                    echo 'Pushing images to Docker Hub...'
                    sh 'docker compose push'
                }
            }
        }
        stage('Clean and Deploy to Dev Environment') {
            steps {
                echo 'Listing images and containers...'
                sh 'docker images'
                sh 'docker compose ps'

                echo 'Cleaning...'
                sh 'docker compose down -v'
                sh 'echo y | docker container prune'
                sh 'docker compose ps'

                echo 'Deploying to Dev Environment...'
                sh 'docker compose up -d'
                sh 'docker compose ps'
                sh 'docker network ls'
                sh 'docker volume ls'
            }
        }
       
    }
    post {
        always {
            echo 'Cleaning...'
            sh 'docker logout'
            cleanWs()
        }
        success {
            echo 'Deployment to Dev Environment is successful!'
        }
        failure {
            echo 'Deployment to Dev Environment failed!'
        }
        unstable {
            echo 'Deployment to Dev Environment is unstable!'
        }
        changed {
            echo 'Deployment to Dev Environment is changed!'
        }
    }
}
