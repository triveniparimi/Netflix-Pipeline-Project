pipeline {
    agent any

    environment {
        // 🔴 CHANGE THESE 3 VALUES
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

        stage('Maven Build') {
            steps {
                // Windows shell
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // These become environment variables, usable as %IMAGE_TAG%
                    env.IMAGE_TAG    = "${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    env.IMAGE_LATEST = "${DOCKER_IMAGE}:latest"
                }

                bat """
                echo Building %IMAGE_TAG%
                docker build -t %IMAGE_TAG% .
                docker tag %IMAGE_TAG% %IMAGE_LATEST%
                """
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                bat """
                docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                docker push %IMAGE_TAG%
                docker push %IMAGE_LATEST%
                docker logout
                """
            }
        }

        stage('Deploy to Kubernetes (EKS)') {
            steps {
                bat """
                kubectl apply -f %K8S_MANIFEST_PATH%/deployment.yml
                kubectl apply -f %K8S_MANIFEST_PATH%/service.yml
                kubectl get deploy
                kubectl get svc
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed: built, pushed and deployed ${DOCKER_IMAGE}"
        }
        failure {
            echo "❌ Pipeline failed – check the stage logs above."
        }
    }
}
