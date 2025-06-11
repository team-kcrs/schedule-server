pipeline {
    agent any

    environment {
        SPRING_PROFILE = "prod"
    }

    stages {
        stage('Prepare .env') {
            steps {
                withCredentials([file(credentialsId: 'wotr-schedule-server-env-file', variable: 'ENV_PATH')]) {
                    cp $ENV_PATH .env
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
                sh '''
                    cp .env src/main/resources/application-prod.properties
                    ./gradlew clean build -Dspring.profiles.active=$SPRING_PROFILE
                '''
            }
        }
    }
}
