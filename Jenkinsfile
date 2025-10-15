pipeline {
  agent any

  environment {
    IMAGE_NAME   = 'demodeploy10'          // Tên image backend
    DOCKER_USER  = 'thyhoang'              // Docker Hub username
    SERVER_USER  = 'ubuntu'                // Tên user SSH của EC2
    SERVER_HOST  = '3.106.239.158'         // Public IP của EC2
  }

  stages {
    stage('Checkout Code') {
      steps {
        // Clone repo từ GitHub (public repo thì không cần credential)
        git branch: 'main', url: 'https://github.com/thy-2004/demodeploy10.git'
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        // Đăng nhập Docker Hub, build & push image
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-cred', 
          usernameVariable: 'DOCKER_USER_NAME', 
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "🔧 Building Docker image..."
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER_NAME" --password-stdin

            # Build image từ Dockerfile.backend
            docker build -t docker.io/$DOCKER_USER/$IMAGE_NAME:latest -f ./Dockerfile.backend .

            echo "📦 Pushing image lên Docker Hub..."
            docker push docker.io/$DOCKER_USER/$IMAGE_NAME:latest

            docker logout
          '''
        }
      }
    }

    stage('Deploy to AWS EC2') {
      steps {
        // Dùng SSH key lưu trong Jenkins credential ID 'deploy-ssh'
        sshagent(credentials: ['deploy-ssh']) {
          sh '''
            echo "🚀 Deploying to EC2..."
            ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_HOST "
              echo '📥 Pull image mới nhất...'
              docker pull docker.io/$DOCKER_USER/$IMAGE_NAME:latest &&

              echo '🧹 Dừng và xóa container cũ nếu có...'
              docker stop $IMAGE_NAME || true &&
              docker rm $IMAGE_NAME || true &&

              echo '⚙️  Chạy container MongoDB nếu chưa có...'
              docker ps -a | grep -q 'mongo' || docker run -d --name mongo -v mongo_data:/data/db -p 27017:27017 mongo &&

              echo '🚀 Khởi chạy container ứng dụng...'
              docker run -d --name $IMAGE_NAME \
                -p 80:5000 \
                --env-file /home/ubuntu/.env \
                --link mongo:mongo \
                docker.io/$DOCKER_USER/$IMAGE_NAME:latest
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
      echo '❌ Deploy thất bại! Kiểm tra lại log trên Jenkins hoặc EC2.'
    }
  }
}
