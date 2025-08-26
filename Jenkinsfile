pipeline {
     agent any

    tools {
        SonarRunnerInstallation 'SonarScanner'   // ✅ Correct type + name
    }

    stages {
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {   // Name from "Configure System"
                    mvn sonar:sonar \
                   -Dsonar.projectKey=myproject \
                   -Dsonar.host.url=http://54.221.49.41:9000 \
                   -Dsonar.login=ed7efca81520f14637ed3ae1a27c932b7ce8709a
                }
            }
        }
    }

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
                      set +e
                      echo "=== Running Trivy scan ==="
                      docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        -v $WORKSPACE:/workspace \
                        aquasec/trivy:0.55.0 image \
                        --exit-code 1 --severity HIGH,CRITICAL \
                        --format table --output /workspace/trivy-report.txt \
                        prasaddablikar16/adservice:latest
                      scan_result=$?
                      echo "Trivy scan finished with exit code: $scan_result"
                      set -e
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push prasaddablikar16/adservice:latest"
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'trivy-report.txt', followSymlinks: false
        }
    }
}
