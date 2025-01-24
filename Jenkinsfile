pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'battleships'  // Base name of the Docker image
        IMAGE_TAG = "battleships:${BUILD_NUMBER}"  // Tag using Jenkins build number
        KUBECONFIG = '/var/lib/jenkins/.kube/config'  // Path to Minikube config 
    }
    stages {
        stage('Clone Repository from GitHub') {
            steps {
                script {
                    // Clone the GitHub repository to Jenkins workspace
                    git 'https://github.com/kshahaji04/battleships.git'
                }
            }
        }
        stage('Build Docker Image in Minikube') {
            steps {
                script {
                    // Use Minikube’s Docker environment
                    sh 'eval $(minikube docker-env)'

                    // Navigate to the project directory and build the Docker image inside Minikube
                    sh "docker build -t ${IMAGE_TAG} ."

                    // Verify image is built inside Minikube
                    sh "docker images | grep battleships"
                }
            }
        }
        stage('Deploy to Minikube') {
            steps {
                script {
                    // Update deployment.yaml to use the new image tag
                    sh "sed -i 's|image: battleships|image: battleships:${BUILD_NUMBER}|' /var/lib/jenkins/workspace/battleships/deployment.yaml"

                    // Apply Deployment and Service configurations
                    sh 'kubectl apply -f /var/lib/jenkins/workspace/battleships/deployment.yaml'
                    sh 'kubectl apply -f /var/lib/jenkins/workspace/battleships/service.yaml'

                    // Verify deployment status
                    sh 'kubectl get pods'
                    sh 'kubectl get svc'
                }
            }
        }
    }
}
