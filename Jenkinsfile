def appname = "hello-newapp"
def repo = "dsohar"  // Replace with your DockerHub username
def appimage = "docker.io/${repo}/${appname}"
def apptag = "${env.BUILD_NUMBER}"
def dockerImage

podTemplate(cloud: 'kubernetes', containers: [
    containerTemplate(
        name: 'jnlp', 
        image: 'jenkins/inbound-agent:latest'
    ),
     containerTemplate(
        name: 'docker', 
        image: 'docker:26-dind', // Use the latest stable DinD image
        privileged: true,      // Essential for Docker daemon to run
        args: '--storage-driver=vfs' // VFS is safest for K8s, though slower
    )], 
  volumes: [
    emptyDirVolume(mountPath: '/var/lib/docker', memory: false) // Q: Why do we need this volume?
  ]) {
    node(POD_LABEL) {
        stage('chackout') {
            container('jnlp') {
            sh '/usr/bin/git config --global http.sslVerify false'
	    checkout scm
          }
        } // end chackout

        stage('Building and Scanning in Parallel') {
            parallel(
                'Build Docker Image': {
                    stage('Build Docker Image') {
                        container('docker') {
                            echo "Building Docker image..."

                            dockerImage = docker.build(
                                "${appimage}:${apptag}",
                                "."
                            )

                            echo "Image Name: ${appimage}:${apptag}"
                        }
                    }
                },

                'Scan Docker Image': {
                    stage('Scan Docker Image') {
                        container('docker') {
                            echo "Scanning..."
                        }
                    }
                }
            )
        }

        stage('push') {
            container('docker') {
              script {
                docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                    dockerImage.push()
                }
              }
            }
        } //end push
    }
}
