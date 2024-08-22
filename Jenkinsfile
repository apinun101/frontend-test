def label = 'aun'  // ใช้ชื่อ Label ที่ถูกต้อง

podTemplate(label: label, containers: [
    containerTemplate(name: 'docker', image: 'google/cloud-sdk:latest', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'kubectl', image: 'bitnami/kubectl:latest', command: 'cat', ttyEnabled: true),
],
volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {

    node(label) {
        def projectId = 'prismatic-crow-429903-r1'
        def region = 'asia-southeast1'
        def repoName = 'devops-repo'
        def imageName = 'my-nextjs-app'
        def imageTag = 'latest'
        def clusterName = 'k8s-cluster01'
        def clusterLocation = 'asia-southeast1' // e.g., 'us-central1-a'

        try {
            stage('Checkout') {
                checkout scm
            }

            stage('Build Docker Image') {
                container('docker') {
                    sh "docker build -t ${imageName}:${imageTag} ."
                }
            }

            stage('Authenticate with Google Cloud') {
                container('docker') {
                    withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh '''#!/bin/bash
                        gcloud auth activate-service-account --key-file="$GOOGLE_APPLICATION_CREDENTIALS"
                        gcloud auth configure-docker asia-southeast1-docker.pkg.dev
                        '''
                    }
                }
            }

            stage('Tag Docker Image') {
                container('docker') {
                    sh "docker tag ${imageName}:${imageTag} ${region}-docker.pkg.dev/${projectId}/${repoName}/${imageName}:${imageTag}"
                }
            }

            stage('Push to Artifact Registry') {
                container('docker') {
                    sh "docker push ${region}-docker.pkg.dev/${projectId}/${repoName}/${imageName}:${imageTag}"
                }
            }

            stage('Deploy to GKE') {
                container('kubectl') {
                    withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh '''#!/bin/bash
                        # Authenticate with Google Cloud
                        gcloud auth activate-service-account --key-file="$GOOGLE_APPLICATION_CREDENTIALS"

                        # Set the project and compute zone
                        gcloud config set project ${projectId}
                        gcloud config set compute/zone ${clusterLocation}

                        # Get credentials for the GKE cluster
                        gcloud container clusters get-credentials ${clusterName}

                        # Apply the Kubernetes deployment
                        kubectl apply -f deployment.yaml
                        '''
                    }
                }
            }
        } finally {
            stage('Clean Workspace') {
                cleanWs()
            }
        }
    }
}
