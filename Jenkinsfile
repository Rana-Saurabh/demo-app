pipeline {

    agent any

    environment {
        AWS_ACCOUNT = "761018892317"
        REGION = "ap-south-1"
        REPO = "jenkins-demo"
        CLUSTER = "default"
        SERVICE = "jenkins-demo-048d"
        TASK_FAMILY = "default-jenkins-demo-048d"
        IMAGE = "${AWS_ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com/${REPO}:${BUILD_NUMBER}"  
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
                set -e
                aws ecr get-login-password --region $REGION |
                docker login --username AWS \
                --password-stdin ${AWS_ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh 'docker tag jenkins-demo:latest $IMAGE'
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $IMAGE'
            }
        }

        stage('Register New Task Definition') {
            steps {
                script {

                    def taskDef = sh(
                        script: "aws ecs describe-task-definition --task-definition ${TASK_FAMILY}",
                        returnStdout: true
                    ).trim()

                    writeFile file: 'taskdef.json', text: taskDef

                    sh '''
                    jq --arg IMAGE "$IMAGE" '
                    .taskDefinition
                    | .containerDefinitions[0].image=$IMAGE
                    | del(
                        .taskDefinitionArn,
                        .revision,
                        .status,
                        .requiresAttributes,
                        .compatibilities,
                        .registeredAt,
                        .registeredBy
                    )' taskdef.json > new-taskdef.json
                    '''

                    env.REVISION = sh(
                        script: "aws ecs register-task-definition --cli-input-json file://new-taskdef.json --query 'taskDefinition.revision' --output text",
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                aws ecs update-service \
                --cluster $CLUSTER \
                --service $SERVICE \
                --task-definition $TASK_FAMILY:$REVISION
                '''
            }
        }
    }
}
