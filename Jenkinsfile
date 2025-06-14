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

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir() // workspace clean
            }
        }
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/team-kcrs/wotr-server.git',
                        credentialsId: 'wotr-git-credentials'
                    ]]
                ])
            }
        }

        stage('Prepare .env') {
            steps {
                withCredentials([file(credentialsId: 'wotr-server-env-file', variable: 'ENV_PATH')]) {
                    sh '''
                      if [ -f "$WORKSPACE/.env" ]; then
                        echo "[DEBUG] Existing .env:"
                        ls -l "$WORKSPACE/.env"
                      fi

                      rm -f "$WORKSPACE/.env"
                      cp "$ENV_PATH" "$WORKSPACE/.env"
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                sh '''
                    chmod +x gradlew
                    export $(cat .env | xargs)
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

        stage('Register Task Definition') {
            steps {
                writeFile file: 'task-definition.json',
                          text: """
                                {
                                    "family": "wotr-fargate-task",
                                    "networkMode": "awsvpc",
                                    "executionRoleArn": "arn:aws:iam::688567260818:role/ecsTaskExecutionRole",
                                    "requiresCompatibilities": ["FARGATE"],
                                    "cpu": "512",
                                    "memory": "1024",
                                    "containerDefinitions": [
                                        {
                                            "name": "wotr-server-fargate",
                                            "image": "$ECR_REPO:$IMAGE_TAG",
                                            "portMappings": [
                                                {
                                                    "containerPort": 8080,
                                                    "protocol": "tcp"
                                                }
                                            ],
                                            "essential": true
                                        }
                                    ]
                                }
                                """

                sh '''
                    aws ecs register-task-definition \
                        --cli-input-json file://task-definition.json \
                        --region $AWS_REGION
                '''
            }
        }
    }
}
