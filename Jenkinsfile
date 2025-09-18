pipeline {
    agent any

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "panthangi"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = "dockerhub"
        GITHUB_CREDENTIALS = "github"
        AWS_REGION = "us-east-1"
    }

    stages {
        stage("Cleanup Workspace") {
            steps { cleanWs() }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: "${GITHUB_CREDENTIALS}",
                    url: 'https://github.com/panthangiEshwary/register-app.git'
            }
        }

        stage("Build Application") {
            steps { sh "mvn clean package -DskipTests" }
        }

        stage("Build & Push Docker Image") {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}",
                                                 usernameVariable: "DOCKER_USER",
                                                 passwordVariable: "DOCKER_PASS")]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest
                        docker logout
                    """
                }
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                build job: 'gitops-register-app-cd',
                      parameters: [string(name: 'IMAGE_TAG', value: "${IMAGE_TAG}")]
            }
        }
    }
}

