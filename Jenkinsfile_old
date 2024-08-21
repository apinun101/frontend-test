import groovy.json.JsonSlurperClassic
def label = "worker-${UUID.randomUUID().toString()}"
def commitSha1() {
  sh 'git rev-parse HEAD > commitSha1'
  def commit = readFile('commitSha1').trim()
  sh 'rm commitSha1'
  commit.substring(0, 11)
}
podTemplate(label: label, containers: [
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {

node(label) {

  stage('Clone repository') {
    checkout scm
  } 

    env.PROJECT_NAME = "frontend"
    env.DOCKER_REGISTRY = "registry.gitlab.com"
    env.REPO = "${env.DOCKER_REGISTRY}/dev/${env.PROJECT_NAME}"
    env.IMAGE_NAME = "${env.REPO}"
    env.COMMITSHA = commitSha1()
    env.TAG_PREFIX = env.BRANCH_NAME
    env.IMAGE_TAG = "${env.TAG_PREFIX}-${commitSha}"
    env.APP_NAME = "test"
    env.KUBE_CREDENTIAL_ID = "kubeconfig"
    env.KUBE_NAMESPACE = "default"
    env.APP_VALUES_FILE = "values-test-prod.yaml"
  
    stage('Build Image') {
			container('docker') {
        script {
           dockerImage = docker.build("${env.IMAGE_NAME}:${env.IMAGE_TAG}", ".")
        }
			}
    }
    // stage('Push Image') {
		// 	container('docker') {
    //     script {
    //         docker.withRegistry("https://${env.DOCKER_REGISTRY}", "gitlab-deploy") {
    //         dockerImage.push()
    //         }
    //     }
    // 	}
		// }
  }
}