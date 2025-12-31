pipeline {
    agent any
    environment {
        // שימי לב: החליפי את המספר למטה במספר החשבון האמיתי שלך מ-AWS
        AWS_ACCOUNT_ID = '992382545251' 
        AWS_DEFAULT_REGION = 'us-east-1'
        IMAGE_REPO_NAME = 'flask-repo'
        IMAGE_TAG = 'latest'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
        EC2_IP = '34.207.187.230'
    }
    stages {
        stage('Login to ECR') {
            steps {
                script {
                    // כאן השינוי: מחקנו את withCredentials והשרת ישתמש ב-IAM Role
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                }
            }
        }
        stage('Build & Push') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_REPO_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${ECR_REGISTRY}/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                    sh "docker push ${ECR_REGISTRY}/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@${EC2_IP} '
                            aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                            docker pull ${ECR_REGISTRY}/${IMAGE_REPO_NAME}:${IMAGE_TAG}
                            docker stop flask-app || true
                            docker rm flask-app || true
                            docker run -d -p 5000:5000 --name flask-app ${ECR_REGISTRY}/${IMAGE_REPO_NAME}:${IMAGE_TAG}
                        '
                    """
                }
            }
        }
    }
}