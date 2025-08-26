pipeline {
    agent any

    stages {
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t prasaddablikar16/adservice:latest ."
                    }
                }
            }
        }

        stage('Trivy Scan Docker Image') {
            steps {
                script {
                    sh '''
                        apt-get update -y
                        apt-get install wget -y
                        wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.55.0_Linux-64bit.deb
                        dpkg -i trivy_0.55.0_Linux-64bit.deb
                        # Scan image for HIGH/CRITICAL vulns (exit 1 if found)
                        trivy image --exit-code 1 --severity HIGH,CRITICAL prasaddablikar16/adservice:latest
                    '''
                }
            }
        }
        
        stage('Push Docker Image') {
            when {
                expression { currentBuild.result == null } // only if Trivy passed
            }
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push prasaddablikar16/adservice:latest"
                    }
                }
            }
        }
    }
}
