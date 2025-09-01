pipeline {
    agent any
    
    environment {
        REGISTRY        = "docker.io"   // or AWS ECR / GCP Artifact / Harbor
        IMAGE_NAME      = "your-dockerhub-username/myapp"
        DOCKER_CREDS    = "docker-credentials-id" // Jenkins credentials (username+password or token)
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/org/repo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Tagging with build number + Git commit hash
                    GIT_COMMIT_SHORT = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    IMAGE_TAG = "${BUILD_NUMBER}-${GIT_COMMIT_SHORT}"
                    
                    echo "Building Docker Image: ${IMAGE_NAME}:${IMAGE_TAG}"
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    // Example using Trivy
                    sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", "${DOCKER_CREDS}") {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push("latest")
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
                sh "docker rmi ${IMAGE_NAME}:latest || true"
            }
        }
    }

    post {
        success {
            echo "✅ Docker Image pushed successfully: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "❌ Build failed. Check logs!"
        }
    }
}
