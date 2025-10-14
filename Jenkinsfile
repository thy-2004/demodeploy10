pipeline {
    agent any

    environment {
        IMAGE_NAME = "demodeploy10"
        DOCKER_USER = "thy2004"              
        SERVER_USER = "ubuntu"
        SERVER_HOST = "15.134.138.22"        
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: 'github-cred', url: 'https://github.com/thy-2004/demodeploy10.git'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat '''
                    docker build -t %DOCKER_USER%/%IMAGE_NAME%:latest .
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    docker push %DOCKER_USER%/%IMAGE_NAME%:latest
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                sshagent (credentials: ['aws-ec2-key']) {
                    bat '''
                    ssh -o StrictHostKeyChecking=no %SERVER_USER%@%SERVER_HOST% ^
                    "docker pull %DOCKER_USER%/%IMAGE_NAME%:latest && ^
                    docker stop %IMAGE_NAME% || true && ^
                    docker rm %IMAGE_NAME% || true && ^
                    docker run -d --name %IMAGE_NAME% -p 80:3000 %DOCKER_USER%/%IMAGE_NAME%:latest"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deploy thành công lên AWS EC2!'
        }
        failure {
            echo '❌ Deploy thất bại!'
        }
    }
}
