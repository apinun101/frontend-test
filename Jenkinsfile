node('docker-agent') {
    podTemplate(inheritFrom: 'docker-agent', customWorkspace: '/home/jenkins/agent',
        yaml: '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:19.03.12
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-socket
  - name: jnlp
    image: jenkins/inbound-agent:latest
    args:
    - ${computer.jnlpmac}
    - ${computer.name}
    resources:
      requests:
        cpu: 100m
        memory: 256Mi
  volumes:
  - name: docker-socket
    hostPath:
      path: /var/run/docker.sock
        ''') {
        node(POD_LABEL) {
            // กำหนด Environment variables
            def PROJECT_ID = 'prismatic-crow-429903-r1'
            def REGION = 'asia-southeast1'
            def REPO_NAME = 'devops'
            def IMAGE_NAME = 'my-nextjs-app'
            def IMAGE_TAG = 'latest'

            try {
                stage('Checkout') {
                    checkout scm
                }

                stage('Build Docker Image') {
                    container('docker') {
                        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    }
                }

                stage('Authenticate with Google Cloud') {
                    withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                    }
                }

                stage('Tag Docker Image') {
                    container('docker') {
                        sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }

                stage('Push to Artifact Registry') {
                    container('docker') {
                        sh "docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            } catch (Exception e) {
                currentBuild.result = 'FAILURE'
                throw e
            } finally {
                stage('Cleanup') {
                    container('docker') {
                        cleanWs()
                    }
                }
            }
        }
    }
}
