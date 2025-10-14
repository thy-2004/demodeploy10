pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('github-cred') // ID credential GitHub
    }

    stage('Clone Code') {
        steps {
            git credentialsId: 'github-cred', url: 'https://github.com/thy-2004/demodeploy10.git', branch: 'main'
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
