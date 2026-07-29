pipeline {
    agent any

    stages {

        stage('Check Docker') {
            steps {
                bat 'docker info'
            }
        }

        stage('Stop Existing Containers') {
            steps {
                bat 'docker compose down'
            }
        }

        stage('Deploy Application') {
            steps {
                bat 'docker compose up -d'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat 'docker ps'
            }
        }

    }
}