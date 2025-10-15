pipeline {
  agent any

  environment {
    DOCKER_USER  = 'thyhoang'
    SERVER_USER  = 'ubuntu'
    SERVER_HOST  = '3.106.239.158'
  }

  stages {
    stage('Checkout Code') {
      steps {
        git branch: 'main', url: 'https://github.com/thy-2004/demodeploy10.git'
      }
    }

    stage('Build & Push Backend + Frontend Images') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-cred', 
          usernameVariable: 'DOCKER_USER_NAME', 
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "🔧 Đăng nhập Docker Hub..."
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER_NAME" --password-stdin

            echo "🐳 Build & push backend..."
            docker build -t docker.io/$DOCKER_USER/proshop-backend:latest -f Dockerfile.backend .
            docker push docker.io/$DOCKER_USER/proshop-backend:latest

            echo "🐳 Build & push frontend..."
            docker build -t docker.io/$DOCKER_USER/proshop-frontend:latest -f frontend/Dockerfile.frontend ./frontend
            docker push docker.io/$DOCKER_USER/proshop-frontend:latest

            docker logout
          '''
        }
      }
    }

    stage('Deploy to EC2 using Docker Compose') {
      steps {
        sshagent(credentials: ['deploy-ssh']) {
          sh '''
            echo "🚀 SSH vào EC2 và deploy..."
            ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_HOST "
              cd /home/ubuntu/demodeploy10 || exit 1

              echo '📥 Pull image mới nhất...'
              docker compose pull

              echo '🧹 Dừng container cũ...'
              docker compose down

              echo '🚀 Chạy lại toàn bộ stack...'
              docker compose up -d

              echo '✅ Hoàn tất deploy!'
            "
          '''
        }
      }
    }
  }

  post {
    success {
      echo '✅ CI/CD thành công — toàn bộ container (mongo + backend + frontend) đã chạy.'
    }
    failure {
      echo '❌ Lỗi deploy — kiểm tra lại log Jenkins hoặc EC2.'
    }
  }
}
