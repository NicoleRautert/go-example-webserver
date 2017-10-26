node {

    checkout scm
    
  def DOCKER_HUB_ACCOUNT = 'rautenic'
  def DOCKER_IMAGE_NAME = 'go-example-webserver'

  echo 'Building Docker image'
  stage('BuildImage') 
  def app = docker.build("${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}", '.')

    echo 'Building Go App'
    stage("build") {
        docker.image("icrosby/jenkins-agent:kube").inside('-u root') {
            sh 'go build' 
        }
    }

    echo 'Testing Go App'
    stage("test") {
        docker.image('icrosby/jenkins-agent:kube').inside('-u root') {
            sh 'go test' 
        }
    }
    
    echo 'Testing Docker image'
    stage("test image") {
        docker.image("${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}").inside {
            sh './test.sh'
        }
    }
    
    stage("Push")
    echo 'Pushing Docker Image'
    docker.withRegistry(' https://localhost:5000', 'docker-hub') {
        app.push("prod")
    }
    
    def K8S_DEPLOYMENT_NAME = 'go-example-webserver'

    stage("Deploy") 
    echo "Deploying image"
    docker.image('smesch/kubectl').inside{
        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
            sh "kubectl --kubeconfig=$KUBECONFIG set image deployment/${K8S_DEPLOYMENT_NAME} ${K8S_DEPLOYMENT_NAME}=${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}"
        }
    }
    
   
}
