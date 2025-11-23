pipeline {
    agent any

    environment {
        // 👇 change this to your Docker Hub username
        DOCKER_IMAGE = "triveniparimi/netflix-app"
        DOCKER_CREDENTIALS_ID = "dockerhub-creds"
        K8S_MANIFEST_PATH = "k8s"
    }

    stages {

        stage('Git Checkout') {
            steps {
                // Jenkins will pull from the same repo where this Jenkinsfile lives
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = "${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    env.IMAGE_LATEST = "${DOCKER_IMAGE}:latest"

                    sh """
                      echo "Building image: ${IMAGE_TAG}"
                      docker build -t ${IMAGE_TAG} .
                      docker tag ${IMAGE_TAG} ${IMAGE_LATEST}
                    """
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push ${IMAGE_TAG}
                      docker push ${IMAGE_LATEST}
                      docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes (EKS)') {
            steps {
                sh """
                  echo 'Deploying manifests to EKS...'
                  kubectl apply -f ${K8S_MANIFEST_PATH}/deployment.yml
                  kubectl apply -f ${K8S_MANIFEST_PATH}/service.yml
                  kubectl get deploy
                  kubectl get svc
                """
            }
        }
    }

    post {
        success {
            echo "✅ Successfully built, pushed, and deployed ${IMAGE_TAG} to EKS."
        }
        failure {
            echo "❌ Pipeline failed. Check the logs for the failing stage."
        }
    }
}
