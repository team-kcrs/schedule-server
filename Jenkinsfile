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
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'wotr-git-credentials',
                    url: 'https://github.com/team-kcrs/wotr-server.git'
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
