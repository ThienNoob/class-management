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

        stage('Set up databases') {
            steps {
                sh '''
                    docker compose -f docker-compose.yml build student-service-mysql
                    docker compose -f docker-compose.yml build auth-service-mysql
                    docker compose -f docker-compose.yml build mongodb  
                    docker compose -f docker-compose.yml build postgresql
                '''
            }
        }

        stage('Set up databases admin tool') {
            steps {
                sh '''
                    docker compose -f docker-compose.yml build auth-phpmyadmin
                    docker compose -f docker-compose.yml build student-phpmyadmin
                    docker compose -f docker-compose.yml build mongo-express
                    docker compose -f docker-compose.yml build pgadmin
                '''
            }
        }

        stage('Build back-end application images') {
            steps {
                sh '''
                    cp class-management-auth-service/Dockerfile .
                    docker compose -f docker-compose.yml build class-mangement-auth-service
                    cp class-management-student-service/Dockerfile .
                    docker compose -f docker-compose.yml build class-management-student-service
                    cp class-management-lecturer-service/Dockerfile .
                    docker compose -f docker-compose.yml build class-management-lecturer-service
                    cp class-management-class-service/Dockerfile .
                    docker compose -f docker-compose.yml build class-management-class-service
                '''
            }
        }

        stage('Build front-end application image') {
            steps {
                sh '''
                    cp class-management-fe/Dockerfile .
                    docker compose -f docker-compose.yml build class-mangement-fe
                '''
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
        
        stage('Scan images with Trivy') {
            steps {
                sh '''
                    trivy image --exit-code 1 --severity HIGH,CRITICAL chucthien03/class-mangement-auth-service
                    trivy image --exit-code 1 --severity HIGH,CRITICAL chucthien03/class-management-student-service
                    trivy image --exit-code 1 --severity HIGH,CRITICAL chucthien03/class-management-lecturer-service
                    trivy image --exit-code 1 --severity HIGH,CRITICAL chucthien03/class-management-class-service
                    trivy image --exit-code 1 --severity HIGH,CRITICAL chucthien03/class-mangement-fe
                '''
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
