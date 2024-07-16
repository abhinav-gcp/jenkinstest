pipeline {
    agent any
    environment {
        PROJECT_ID = 'sukrit-singh-426716'
        CLUSTER_NAME = 'my-gke-cluster'
        LOCATION = 'us-east1'
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
                    myapp = docker.build("${DOCKER_IMAGE_NAME}:${env.BUILD_ID}")
                }
            }
        }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }        
        stage('Deploy to GKE') {
            steps{
                sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' deployment.yaml"
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS_FILE')]) {
                    withEnv(["GOOGLE_APPLICATION_CREDENTIALS=${GOOGLE_APPLICATION_CREDENTIALS_FILE}"]) {
                        step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', verifyDeployments: true])
                    }
                }
            }
        }
    }    
}
