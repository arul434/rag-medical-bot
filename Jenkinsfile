pipeline {
    agent any

    environment {
        REGISTRY = 'localhost:5000'
        IMAGE_NAME = 'medical-chatbot'
        TAG = 'latest'
        FULL_IMAGE = "${REGISTRY}/${IMAGE_NAME}:${TAG}"
        DEPLOYMENT_NAME = 'medical-chatbot-app'
        NAMESPACE = 'default'
    }

    stages {
        stage('Clone GitHub Repo') {
            steps {
                script {
                    echo 'Cloning GitHub repo...'
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-token', url: 'https://github.com/arul434/rag-medical-bot.git']])                }
            }
        }

        stage('Build, Scan, and Push') {
            steps {
                script {
                    echo "Building image: ${FULL_IMAGE}"
                    sh "docker build -t ${FULL_IMAGE} ."

                    echo "Scanning with Trivy..."
                    // Scan for vulnerabilities but don't fail the build (exit-code 0 or || true)
                    sh "trivy image --severity HIGH,CRITICAL --format table ${FULL_IMAGE} || true"

                    echo "Pushing to Local Registry..."
                    sh "docker push ${FULL_IMAGE}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying to Namespace: ${NAMESPACE}"
                    // Trigger a rollout restart to pull the new image
                    sh "kubectl rollout restart deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE}"
                    
                    // Verify rollout status
                    sh "kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE}"
                }
            }
        }
    }
}
