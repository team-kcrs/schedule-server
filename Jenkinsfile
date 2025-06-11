pipeline {
    agent any
    environment {
        // Jenkins Credentials ID
        ENV_FILE = credentials('wotr-schedule-server-env')
    }
    stages {
        stage('Write .env') {
            steps {
                writeFile file: '.env', text: ENV_FILE
            }
        }
        stage('Checkout') {
            steps {
                git branch: 'main',
                    // Jenkins Credentials ID
                    credentialsId: 'wotr-schedule-server-credential',
                    url: 'https://github.com/team-kcrs/schedule-server.git'
            }
        }
        stage('Build') {
            steps {
                sh 'chmod +x gradlew'
                sh '''
                    cp .env src/main/resources/application-prod.properties
                    ./gradlew clean build -Dspring.profiles.active=prod
                '''
            }
        }
    }
}