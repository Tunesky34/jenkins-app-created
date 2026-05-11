// jenkins-docker pipeline scriipt

pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
      steps {
        sh "docker build -t myapp:${BUILD_NUMBER} ."
      }
    }

    stage('Test') {
      steps {
        sh """
          docker run -d --name test_app -p 4000:3000 myapp:${BUILD_NUMBER}
          sleep 5
          curl http://localhost:4000
          docker stop test_app || true
          docker rm test_app || true
        """
      }
    }

    stage('Deploy') {
      steps {
        sh """
          docker stop myapp || true
          docker rm myapp || true
          docker run -d --name myapp -p 80:3000 myapp:${BUILD_NUMBER}
        """
      }
    }
  }

  post {
    failure {
      echo 'Deployment failed!'
    }
    success {
      echo "Deployment succeeded: myapp:${BUILD_NUMBER} is running."
    }
  }
}
