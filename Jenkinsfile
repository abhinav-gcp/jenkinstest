pipeline {
    agent any
    environment {
        PROJECT_ID = 'amplified-asset-426508-e7'
        CLUSTER_NAME = 'my-gke-cluster'
        LOCATION = 'us-central1-a'
        CREDENTIALS_ID = 'amplified-asset-426508-e7'
        BUILD_ID = "${env.BUILD_NUMBER}"
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
                    myapp = docker.build("abhinavsingh8477/hello:${BUILD_ID}")
                }
            }
        }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerID') {
                            myapp.push("latest")
                            myapp.push("${BUILD_ID}")
                    }
                }
            }
        }        
        stage('Deploy to GKE') {
            steps{
                sh "sed -i 's|hello:latest|hello:${BUILD_ID}|g' deployment.yaml"
                withCredentials([file(credentialsId: 'gke-deploy-credentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: 'gke-deploy-credentials', verifyDeployments: true])
                }
            }
        }
    }
}