pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '940482429259.dkr.ecr.us-east-1.amazonaws.com/my-ecr-repo'
        IMAGE_NAME = 'my-ecr-repo'
        CLUSTER_NAME = 'my-cluster'
        SERVICE_NAME = 'my-app-service'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Vinayvurukutigithub/Docker-fargate.git'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh 'docker build -t $IMAGE_NAME .'
                    sh 'docker tag $IMAGE_NAME:latest $ECR_REPO:latest'
                }
            }
        }

        stage('ECR Login & Push') {
            steps {
                script {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    docker push $ECR_REPO:latest
                    '''
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('terraform') {
                    sh '''
                    terraform init
                    terraform apply -auto-approve
                    '''
                }
            }
        }

        stage('Deploy New ECS Task') {
            steps {
                script {
                    sh '''
                    aws ecs update-service \
                        --cluster $CLUSTER_NAME \
                        --service $SERVICE_NAME \
                        --force-new-deployment \
                        --region $AWS_REGION
                    '''
                }
            }
        }
    }
}
