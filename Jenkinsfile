pipeline {
  agent any

  environment {
    IMAGE_NAME   = 'demodeploy10'          // Tên image
    DOCKER_USER  = 'thyhoang'              // Docker Hub username
    SERVER_USER  = 'ubuntu'                // SSH user EC2
    SERVER_HOST  = '3.106.239.158'         // IP EC2
  }

  stages {
    stage('Checkout Code') {
      steps {
        // Repo public, không cần credential
        git branch: 'main', url: 'https://github.com/thy-2004/demodeploy10.git'
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-cred', 
          usernameVariable: 'DOCKER_USER_NAME', 
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER_NAME" --password-stdin

            # Build image backend bằng Dockerfile.backend
            docker build -t docker.io/$DOCKER_USER/$IMAGE_NAME:latest -f ./Dockerfile.backend .

            # Push image lên Docker Hub
            docker push docker.io/$DOCKER_USER/$IMAGE_NAME:latest

            docker logout
          '''
        }
      }
    }

    stage('Deploy to AWS EC2') {
      steps {
        sshagent(credentials: ['deploy-ssh']) {
          sh '''
            ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_HOST "
              docker pull docker.io/$DOCKER_USER/$IMAGE_NAME:latest &&
              docker stop $IMAGE_NAME || true &&
              docker rm $IMAGE_NAME || true &&
              docker run -d --name $IMAGE_NAME -p 80:3000 docker.io/$DOCKER_USER/$IMAGE_NAME:latest
            "
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
