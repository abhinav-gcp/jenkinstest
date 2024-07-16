pipeline {
    agent any
    environment {
        PROJECT_ID = 'sukrit-singh-426716'
        CLUSTER_NAME = 'my-gke-cluster'
        LOCATION = 'us-east1'
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
                sh "sed -i '|hello:latest|hello:${BUILD_ID}|g' deployment.yaml"
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS_FILE')]) {
                    withEnv(["GOOGLE_APPLICATION_CREDENTIALS=${GOOGLE_APPLICATION_CREDENTIALS_FILE}"]) {
                        sh "gcloud auth activate-service-account --key-file ${GOOGLE_APPLICATION_CREDENTIALS_FILE}"
                        sh "gcloud config set project ${PROJECT_ID}"
                        sh "gcloud config set compute/region ${LOCATION}"
                        sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${LOCATION} --project ${PROJECT_ID}"
                        // Install kubectl
                        sh "curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
                        sh "chmod +x kubectl"
                        sh "sudo mv kubectl /usr/local/bin/kubectl"
                        // Deploy to GKE
                        sh "kubectl apply -f deployment.yaml"
                    }
                }
            }
        }
    }
}
