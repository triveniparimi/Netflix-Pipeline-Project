pipeline {
    agent any

    environment {
        DOCKER_IMAGE          = "triveniparimi/netflix-app"
        DOCKER_CREDENTIALS_ID = "dockerhub-creds"
        K8S_MANIFEST_PATH     = "k8s"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                // Windows shell = bat, not sh
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG    = "${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    env.IMAGE_LATEST = "${DOCKER_IMAGE}:latest"
                }

                // Use Windows batch script
                bat """
                echo Building image %IMAGE_TAG%
                docker build -t %IMAGE_TAG% .
                docker tag %IMAGE_TAG% %IMAGE_LATEST%
                """
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    docker push %IMAGE_TAG%
                    docker push %IMAGE_LATEST%
                    docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes (EKS)') {
            steps {
                bat """
                kubectl apply -f k8s/deployment.yml
                kubectl apply -f k8s/service.yml
                kubectl get deploy
                kubectl get svc
                """
            }
        }
    }
}
