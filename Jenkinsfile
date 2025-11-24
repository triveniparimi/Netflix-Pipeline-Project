pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        DOCKER_IMAGE = "triveniparimi/netflix-app"
        DOCKER_USER  = "triveniparimi"
        DOCKER_PASS  = "Remember2023"
        K8S_MANIFEST_PATH = "k8s"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                bat "mvn clean package -DskipTests"
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %DOCKER_IMAGE% ."
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                bat "docker push %DOCKER_IMAGE%"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat "kubectl apply -f k8s\\deployment.yml"
                bat "kubectl apply -f k8s\\service.yml"
            }
        }
    }
}
