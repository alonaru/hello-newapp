
def appname = "hello-newapp"
def repo = "alonaru"  // Replace with your DockerHub username
def artifactory = "docker.io" 
def appimage = "docker.io/${repo}/${appname}"
def apptag = "${env.BUILD_NUMBER}"
def deploy = true



podTemplate(containers: [
      containerTemplate(name: 'jnlp', image: 'jenkins/inbound-agent', ttyEnabled: true),
      containerTemplate(name: 'docker', image: 'gcr.io/kaniko-project/executor:debug-v0.19.0', command: "/busybox/cat", ttyEnabled: true),
      containerTemplate(name: 'helm', image: 'alpine/helm:3.13.1', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
     configMapVolume(mountPath: '/kaniko/.docker/', configMapName: 'docker-cred')
  ])  {
    node(POD_LABEL) {
        stage('chackout') {
            container('jnlp') {
            sh '/usr/bin/git config --global http.sslVerify false'
	    checkout scm
          }
        } // end chackout

        stage('build') {
            container('kaniko') {
                echo "Building docker image with kaniko..."
	            sh "/kaniko/executor --context . --dockerfile Dockerfile --destination=${appimage} --insecure --skip-tls-verify"
            }
        } //end build
        stage('helm install') {
            container('helm') {
                echo "Running Helm install..."
                sh "curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3"
                sh "chmod 700 get_helm.sh"
                sh "./get_helm.sh"
            }
        }
        stage('deploy') {
            container('docker') {
	      if (deploy) {
                echo "***** Doing some deployment stuff *********"
             }  else {
                echo "***** NO DEPLOY - Doing somthing else. Testing? *********"
             }
           }
        } //end deploy
    }
    /*
    post {
        always {
            echo 'Pipeline completed!'
        }
        success {
            echo 'Pipeline succeeded!'
            echo 'Send sucess email'
            echo 'Notify CMDB'
        }
        failure {
            echo 'Pipeline failed!'
            echo 'send error email'
        }
    }
    */
}