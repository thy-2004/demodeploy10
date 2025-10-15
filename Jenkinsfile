pipeline {
  agent any

  environment {
    IMAGE_NAME   = 'demodeploy10'
    DOCKER_USER  = 'thyhoang'         // khớp credential Docker Hub
    SERVER_USER  = 'ubuntu'
    SERVER_HOST  = '15.134.138.22'
  }

  stages {
    stage('Checkout Code') {
      steps {
        // Repo public -> không cần credential
        git branch: 'main', url: 'https://github.com/thy-2004/demodeploy10.git'
        // Nếu repo private thì dùng:
        // git branch: 'main', credentialsId: 'github-token', url: 'https://github.com/thy-2004/demodeploy10.git'
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-cred',
                         usernameVariable: 'DOCKER_USER_NAME',
                         passwordVariable: 'DOCKER_PASS')]) {
          bat '''
          echo %DOCKER_PASS% | docker login -u %DOCKER_USER_NAME% --password-stdin

          REM =======================
          REM CHỌN 1 TRONG 2 CÁCH:
          REM A) Nếu repo CÓ Dockerfile ở GỐC:
          REM docker build -t docker.io/%DOCKER_USER%/%IMAGE_NAME%:latest .

          REM B) Nếu Dockerfile NẰM CHỖ KHÁC (ví dụ .\\Dockerfile.backend + code trong .\\backend):
          REM docker build -t docker.io/%DOCKER_USER%/%IMAGE_NAME%:latest -f .\\Dockerfile.backend .\\backend
          REM =======================

          docker push docker.io/%DOCKER_USER%/%IMAGE_NAME%:latest
          docker logout
          '''
        }
      }
    }

    stage('Deploy to AWS EC2') {
      steps {
        // Cần plugin "SSH Agent"
        sshagent(credentials: ['deploy-ssh']) {
          bat '''
          ssh -o StrictHostKeyChecking=no %SERVER_USER%@%SERVER_HOST% ^
            "docker pull docker.io/%DOCKER_USER%/%IMAGE_NAME%:latest && ^
             docker stop %IMAGE_NAME% || true && ^
             docker rm %IMAGE_NAME% || true && ^
             docker run -d --name %IMAGE_NAME% -p 80:3000 docker.io/%DOCKER_USER%/%IMAGE_NAME%:latest"
          '''
        }
      }
    }
  }

  post {
    success { echo '✅ Deploy thành công lên AWS EC2!' }
    failure { echo '❌ Deploy thất bại!' }
  }
}
