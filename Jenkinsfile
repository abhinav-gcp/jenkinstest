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
        GKE_CREDENTIALS_ID = 'gcp-key'
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
                withCredentials([file(credentialsId: GKE_CREDENTIALS_ID, variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${LOCATION} --project ${PROJECT_ID}"
                    sh "sed -i '/${DOCKER_IMAGE_NAME}:latest/${DOCKER_IMAGE_NAME}:${BUILD_ID}/g' deployment.yaml"
                    kubernetesDeploy(
                        configs: 'deployment.yaml',
                        enableConfigSubstitution: true,
                        kubeconfigId: GKE_CREDENTIALS_ID,
                        clusterUrl: "${CLUSTER_NAME}.gke.${LOCATION}.cloud.google.com",
                        clusterCertificate: '',
                        namespace: 'default', 
                        disableInlineConfig: false
                    )
                }
            }
        }
    }
}
