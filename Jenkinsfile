pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('github-cred') // ID credential GitHub
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main', credentialsId: 'github-cred', url: 'https://github.com/thy-2004/proshop-v2.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Run Containers') {
            steps {
                sh 'docker compose up -d'
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker image prune -f'
            }
        }
    }
}
