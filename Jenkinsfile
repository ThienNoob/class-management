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
        stage('Set up database') {
            steps {
                echo 'Building mysql image...'
                sh 'docker compose build mysql'
                echo 'Building phpmyadmin image...'
                sh 'docker compose build phpmyadmin'
            }
        }
        stage('Build back-end application image') {
            steps {
                echo 'student-backend is being built...'
                sh 'docker compose build student-backend'
            }
        }
        stage('Build front-end application image') {
            steps {
                echo 'student-fronend is being built...'
                sh 'docker compose build student-frontend'
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
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-token', namespace: 'jenkins', restrictKubeConfigAccess: false, serverUrl: 'https://ec2-52-221-253-100.ap-southeast-1.compute.amazonaws.com:6443') {
                    echo 'Deploying to Kubernetes...'
                    sh 'kubectl apply -f secrets.yaml'
                    sh 'kubectl apply -f configMap.yaml'
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                    echo 'Checking deployment status...'
                    sh 'kubectl get svc'
                    sh 'kubectl get pods'
                }
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