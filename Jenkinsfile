pipeline {
    agent any
    environment {
        ECR_REGISTRY = "877540899436.dkr.ecr.us-east-1.amazonaws.com"
        APP_REPO_NAME = "hasan05/to-do-webapp"
        KUBE_MASTER_IP = "44.202.50.190"
        ANS_KEYPAIR = "secondkey" // Add your ANS_KEYPAIR value here
    }
    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build --force-rm -t "$ECR_REGISTRY/$APP_REPO_NAME:latest" .'
                sh 'docker image ls'
            }
        }
        stage('Push Image to ECR Repo') {
            steps {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin "$ECR_REGISTRY"'
                sh 'docker push "$ECR_REGISTRY/$APP_REPO_NAME:latest"'
            }
        }
        stage('Create Infrastructure') {
            steps {
                echo 'Creating infrastructure using Terraform'
                dir('infrastructure') {
                    sh """
                        sed -i 's/secondkey/${ANS_KEYPAIR}/g' main.tf
                        terraform init
                        terraform apply -auto-approve -no-color
                                                                      
                    """
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes'
                    withKubeConfig([credentialsId: 'kube-config', serverUrl: "https://$44.202.50.190"]) {
                        sh 'kubectl apply -f ./* --namespace my-namespace'
                    }
                }
            }
        }
    }  
}