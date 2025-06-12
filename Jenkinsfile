pipeline {
    agent any

    environment {
        SPRING_PROFILE = "prod"

        // AWS ECR
        AWS_REGION = "ap-northeast-2"
        ECR_REPO = "688567260818.dkr.ecr.ap-northeast-2.amazonaws.com/wotr-ecr"
        IMAGE_TAG = "latest"
        IMAGE_NAME="jenkins-with-awscli"
    }

    stages {
        stage('Prepare .env') {
            steps {
                withCredentials([file(credentialsId: 'wotr-schedule-server-env-file', variable: 'ENV_PATH')]) {
                    sh 'cp $ENV_PATH .env'
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'wotr-schedule-server-credential',
                    url: 'https://github.com/team-kcrs/schedule-server.git'
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x gradlew'

                // dash 에서 source 대신 . (dot) 사용
                sh '''
                    set -a
                    . .env
                    set +a

                    ./gradlew clean build -Dspring.profiles.active=$SPRING_PROFILE
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME .
                    docker tag $IMAGE_NAME:latest $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'wotr-aws-credentials'
                ]]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION \
                        | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh 'docker push $ECR_REPO:$IMAGE_TAG'
            }
        }
    }
}
