pipeline {
    agent any
    stages {
        stage('Git Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/team-kcrs/schedule-server.git'
            }
        }
        stage('Build Backend') {
            steps {
                dir('./') {
                    sh './gradlew clean build'
                }
            }
        }
    }
}