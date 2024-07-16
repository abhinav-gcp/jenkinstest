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
        CREDENTIALS_ID = 'gcp-key'
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
                sh "sed -i \"s|${DOCKER_IMAGE_NAME}:latest|hello:${BUILD_ID}|g\" deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', 
                      projectId: env.PROJECT_ID, 
                      clusterName: env.CLUSTER_NAME, 
                      location: env.LOCATION, 
                      manifestPattern: 'deployment.yaml', 
                      credentialsId: env.CREDENTIALS_ID, 
                      verifyDeployments: true])
            }
        }
    }
}
