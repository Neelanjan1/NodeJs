  pipeline {
    agent any
    environment {
    AWS_ACCOUNT_ID="867366814810"
    AWS_DEFAULT_REGION="us-east-1"
    IMAGE_REPO_NAME="demo-ecr"
    IMAGE_TAG="${BUILD_NUMBER}"
    REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
  }

    stages {
        stage('Logging into AWS ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
            }
        }
        stage('Cloning Git') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Neelanjan1/NodeJs.git']])

            }
        }
        // Building Docker images
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        }

        // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{
                script {
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy container') {
            steps{
                script {
                    def dockerStopCmd = 'docker rm --force nodeweb'
                    sshagent(credentials: ['app']) {
                        sh "ssh -vvv -o StrictHostKeyChecking=no -T ubuntu@54.196.157.193  ${dockerStopCmd}"
                    }
                    def dockerCmd = 'docker run -itd -p 8080:8080 --name nodeweb neelanjan1/node-web-app'
                    sshagent(credentials: ['app']) {
                        sh "ssh -vvv -o StrictHostKeyChecking=no -T ubuntu@54.196.157.193  ${dockerCmd}"
                    }
                }
            }
        }
    }
}