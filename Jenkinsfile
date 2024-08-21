pipeline {
    agent {
        kubernetes {
            // Define the Pod Template using Kubernetes plugin
            label 'docker-agent'
            customWorkspace '/home/jenkins/agent'
            podTemplate(
                label: 'docker-agent',
                containers: [
                    containerTemplate(
                        name: 'docker',
                        image: 'docker:19.03.12', // Use a Docker image with Docker CLI
                        command: 'cat',
                        ttyEnabled: true,
                        volumeMounts: [
                            // Mount Docker socket to allow Docker commands
                            mountPath: '/var/run/docker.sock',
                            name: 'docker-socket'
                        ]
                    ),
                    containerTemplate(
                        name: 'jnlp',
                        image: 'jenkins/inbound-agent:latest',
                        args: '${computer.jnlpmac} ${computer.name}',
                        resourceRequestCpu: '100m',
                        resourceRequestMemory: '256Mi'
                    )
                ],
                volumes: [
                    hostPathVolume(
                        mountPath: '/var/run/docker.sock',
                        hostPath: '/var/run/docker.sock'
                    )
                ]
            )
        }
    }

    environment {
        PROJECT_ID = 'prismatic-crow-429903-r1'
        REGION = 'asia-southeast1'
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
                    sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
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
                    sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}'
                }
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                container('docker') {
                    sh 'docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}'
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
