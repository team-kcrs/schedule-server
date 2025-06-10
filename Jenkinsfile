pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'wonnx-full-privileges',
                    url: 'https://github.com/team-kcrs/schedule-server.git'
            }
        }
        stage('Build Backend') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew clean build'
            }
        }
    }
}