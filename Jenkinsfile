def label = 'aun'  // ใช้ชื่อ Label ที่ถูกต้อง

podTemplate(
    label: label,
    containers: [
        containerTemplate(
            name: 'docker',
            image: 'docker:19.03.12',  // ใช้ Docker image ที่ต้องการ
            command: 'cat',
            ttyEnabled: true,
            volumeMounts: [
                // Mount Docker socket for Docker operations
                volumeMount(mountPath: '/var/run/docker.sock', name: 'docker-socket')
            ]
        )
    ],
    volumes: [
        hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock', name: 'docker-socket')
    ]
) {
    node(label) {
        def projectId = 'prismatic-crow-429903-r1'
        def region = 'asia-southeast1'
        def repoName = 'devops'
        def imageName = 'my-nextjs-app'
        def imageTag = 'latest'

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
                withCredentials([file(credentialsId: 'gcp-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh "gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS"
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
        } finally {
            stage('Clean Workspace') {
                cleanWs()
            }
        }
    }
}
