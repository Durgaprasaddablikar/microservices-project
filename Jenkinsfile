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
                    docker run --rm \
                      -v /var/run/docker.sock:/var/run/docker.sock \
                      aquasec/trivy:0.55.0 image \
                      --exit-code 1 --severity HIGH,CRITICAL \
                      prasaddablikar16/adservice:latest
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
