pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16' // Ensure 'node16' is configured in Jenkins under Manage Jenkins > Global Tool Configuration
    }
    environment {
        SCANNER_HOME = tool 'sonar'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }


        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/harshitsahu2311/Amazon-Prime-Clone-App.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=prime-clone \
                        -Dsonar.projectKey=prime-clone
                    '''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred'
                }
            }
        }
        stage ("Trivy File Scan") {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image tagged as 'prime-clone:latest'
                    sh "docker build -t prime-clone:latest ."
                }
            }
        }
        stage ("Trivy Image Scan") {
            steps {
                sh "trivy image prime-clone:latest"
            }
        }
        stage('Tag & Push to DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker tag prime-clone:latest harshitsahu2311/prime-clone:latest"
                        sh "docker push harshitsahu2311/prime-clone:latest"
                    }
                }
            }
        }
        stage ("Deploy to Kubernetes") {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kuber-cred', variable: 'KUBECONFIG_FILE')]) {
                        sh """
                            # Set the KUBECONFIG environment variable
                            export KUBECONFIG=${KUBECONFIG_FILE}

                            # Navigate to the Kubernetes manifests directory
                            cd Kubernetes

                            # Apply all Kubernetes manifests
                            kubectl apply -f .

                        """
                    }
                }
            }

        }
    }

}
post {
    success {
            echo '✅ Deployment successful!'
            // Optional: Add notifications (e.g., email, Slack) here
    }
    failure {
            echo '❌ Deployment failed.'
            // Optional: Add notifications (e.g., email, Slack) here
    }
}
