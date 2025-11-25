pipeline {
    agent any

    tools {
        // Must match the name you configured under:
        // Manage Jenkins -> Tools -> Maven installations
        maven 'maven'
    }

    environment {
        DOCKER_IMAGE      = "triveniparimi/netflix-app"   // your Docker Hub repo
        DOCKER_USER       = "triveniparimi"
        DOCKER_PASS       = "YOUR_DOCKER_PASSWORD"
        K8S_MANIFEST_PATH = "k8s"                         // folder with deployment.yml & service.yml
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
                bat """
                docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .
                docker tag %DOCKER_IMAGE%:%BUILD_NUMBER% %DOCKER_IMAGE%:latest
                """
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %DOCKER_IMAGE%:%BUILD_NUMBER%
                    docker push %DOCKER_IMAGE%:latest
                    docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes (Minikube)') {
            steps {
                bat """
                kubectl apply -f %K8S_MANIFEST_PATH%\\deployment.yml
                kubectl apply -f %K8S_MANIFEST_PATH%\\service.yml
                kubectl get pods
                kubectl get svc
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully."
        }
        failure {
            echo "❌ Pipeline failed – check the stage logs above."
        }
    }
}
