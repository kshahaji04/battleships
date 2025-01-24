pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'nodejs-shopping'  // Base name of the Docker image
        IMAGE_TAG = "nodejs-shopping:${BUILD_NUMBER}"  // Tag using Jenkins build number
        KUBECONFIG = '/var/lib/jenkins/.kube/config'  // Path to Minikube config 
    }
    stages {
        stage('Clone Repository from GitHub') {
            steps {
                script {
                    // Clone the GitHub repository to Jenkins workspace
                    git 'https://github.com/kshahaji04/nodejs-shopping.git'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Navigate to the project directory and build the Docker image with a new tag
                    dir('') {
                        sh "docker build -t ${IMAGE_TAG} ."
                    }
                }
            }
        }
        stage('Deploy to Minikube') {
            steps {
                script {
                    // Update deployment.yaml to use the new image tag
                    sh "sed -i 's|image: nodejs-shopping|image: nodejs-shopping:${BUILD_NUMBER}|' /var/lib/jenkins/workspace/Test4-G/deployment.yaml"
        
                    // Apply both Deployment and Service configurations to Minikube
                    sh 'kubectl apply -f /var/lib/jenkins/workspace/Test4-G/deployment.yaml'
                    sh 'kubectl apply -f /var/lib/jenkins/workspace/Test4-G/service.yaml'

                    // Verify deployment status
                    sh 'kubectl get pods'
                    sh 'kubectl get svc'
                }
            }
        }
    }
}
