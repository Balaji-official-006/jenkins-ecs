pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "017540984476.dkr.ecr.us-east-1.amazonaws.com/jenkins-ecs-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Balaji-official-006/jenkins-ecs.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t $ECR_REPO:$IMAGE_TAG .
                  docker tag $ECR_REPO:$IMAGE_TAG $ECR_REPO:latest
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: AWS_REGION) {
                    sh '''
                      aws ecr get-login-password \
                      | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                  docker push $ECR_REPO:$IMAGE_TAG
                  docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy to ECS') {
            steps {
                withAWS(credentials: 'aws-creds', region: AWS_REGION) {
                    sh '''
                      aws ecs update-service \
                        --cluster my-ecs-cluster \
                        --service my-ecs-service \
                        --force-new-deployment
                    '''
                }
            }
        }
    }
}
