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
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main',
                    credentialsId: "${GITHUB_CREDENTIALS}",
                    url: 'https://github.com/panthangiEshwary/register-app.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package -DskipTests"
            }
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
                script {
                    sh "curl -v -k --user aakash:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://ec2-15-207-107-126.ap-south-1.compute.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
        }
    }
}

