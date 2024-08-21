pipeline {
    agent {
        kubernetes {
            label 'docker-agent'
            defaultContainer 'jnlp'
        }
    }

    environment {
        PROJECT_ID = 'prismatic-crow-429903-r1'
        REGION = 'asia-southeast1' // e.g., us-central1
        REPO_NAME = 'devops'
        IMAGE_NAME = 'my-nextjs-app'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                    }
                }
            }
        }

        stage('Authenticate with Google Cloud') {
            steps {
                container('docker') {
                    withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                    }
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                container('docker') {
                    script {
                        sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}'
                    }
                }
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                container('docker') {
                    script {
                        sh 'docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}'
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
