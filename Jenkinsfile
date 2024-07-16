pipeline {
    agent any
    environment {
        PROJECT_ID = 'amplified-asset-426508-e7'
        CLUSTER_NAME = 'my-gke-cluster'
        LOCATION = 'us-central1-a'
        BUILD_ID = "${env.BUILD_NUMBER}"
        DOCKER_REGISTRY = 'https://registry.hub.docker.com'
        DOCKER_IMAGE_NAME = 'abhinavsingh8477/hello'
        DOCKER_CREDENTIALS_ID = 'dockerID'
        GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-key')
    }

    stages {
        stage("Checkout code") {
            steps {
                checkout scm
            }
        }
        stage("Build image") {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE_NAME}:${BUILD_ID}")
                }
            }
        }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry(DOCKER_REGISTRY, DOCKER_CREDENTIALS_ID) {
                        docker.image("${DOCKER_IMAGE_NAME}:${BUILD_ID}").push("latest")
                        docker.image("${DOCKER_IMAGE_NAME}:${BUILD_ID}").push("${BUILD_ID}")
                    }
                }
            }
        }
        stage('Deploy to GKE') {
            steps {
                sh "sed -i 's|hello:latest|hello:${BUILD_ID}|g' deployment.yaml"
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS_FILE')]) {
                    withEnv(["GOOGLE_APPLICATION_CREDENTIALS=${GOOGLE_APPLICATION_CREDENTIALS_FILE}"]) {
                        sh "gcloud auth activate-service-account --key-file ${GOOGLE_APPLICATION_CREDENTIALS_FILE}"
                        sh "gcloud config set project ${PROJECT_ID}"
                        sh "gcloud config set compute/zone ${LOCATION}"
                        sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${LOCATION} --project ${PROJECT_ID}"
                        sh "kubectl apply -f deployment.yaml"
                    }
                }
            }
        }
    }
}