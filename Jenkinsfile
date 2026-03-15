pipeline {

    agent any

    environment {
        AWS_ACCOUNT = "761018892317"
        REGION = "ap-south-1"
        REPO = "jenkins-demo"
        CLUSTER = "jenkins-demo-cluster"
        SERVICE = "jenkins-demo-service"
        TASK_FAMILY = "default-jenkins-demo-048d"
        IMAGE = "$AWS_ACCOUNT.dkr.ecr.$REGION.amazonaws.com/$REPO:latest"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t jenkins-demo:latest .'
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

        stage('Tag Image') {
            steps {
                sh '''
                docker tag jenkins-demo:latest $IMAGE
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                docker push $IMAGE
                '''
            }
        }

        stage('Register New Task Definition') {
            steps {
                sh '''
                TASK_DEF=$(aws ecs describe-task-definition \
                --task-definition $TASK_FAMILY)

                NEW_TASK_DEF=$(echo $TASK_DEF | jq \
                --arg IMAGE "$IMAGE" \
                '.taskDefinition |
                {
                    family: .family,
                    networkMode: .networkMode,
                    containerDefinitions: (.containerDefinitions | map(.image=$IMAGE)),
                    requiresCompatibilities: .requiresCompatibilities,
                    cpu: .cpu,
                    memory: .memory
                }')

                aws ecs register-task-definition \
                --cli-input-json "$NEW_TASK_DEF"
                '''
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh '''
                aws ecs update-service \
                --cluster $CLUSTER \
                --service $SERVICE \
                --force-new-deployment
                '''
            }
        }

    }
}
