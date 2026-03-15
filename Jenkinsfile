pipeline {

    agent any

    environment {
        AWS_ACCOUNT = "761018892317"
        REGION = "ap-south-1"
        REPO = "jenkins-demo"
    }

    stages {

        stage('Clone Repo') {
            steps {
                echo "Code already checked out by Jenkins"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t jenkins-demo .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $REGION \
                | docker login --username AWS \
                --password-stdin $AWS_ACCOUNT.dkr.ecr.$REGION.amazonaws.com
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker tag jenkins-demo:latest \
                $AWS_ACCOUNT.dkr.ecr.$REGION.amazonaws.com/$REPO:latest

                docker push \
                $AWS_ACCOUNT.dkr.ecr.$REGION.amazonaws.com/$REPO:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                aws ecs update-service \
                --cluster jenkins-demo-cluster \
                --service jenkins-demo-service \
                --force-new-deployment
                '''
            }
        }

    }
}
