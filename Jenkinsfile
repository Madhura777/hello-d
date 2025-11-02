pipeline {
  agent any

  environment {
    IMAGE_NAME = "hello-d"
    IMAGE_TAG  = "latest"
    CONTAINER_NAME = "hello-d"
    APP_PORT = "5001"   // Flask app runs inside container on port 5001
  }

  stages {

    stage('Checkout') {
      steps {
        echo '📂 Checking out source code...'
      }
    }

    stage('Build Docker Image') {
      steps {
        echo '🐳 Building Docker image...'
        sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
      }
    }

    stage('Run Container') {
      steps {
        echo '🚀 Running the container...'
        sh '''
          if [ "$(docker ps -q -f name=${CONTAINER_NAME})" ]; then
            echo "Stopping and removing old container..."
            docker rm -f ${CONTAINER_NAME} || true
          fi
          echo "Starting new container..."
          docker run -d -p 5001:${APP_PORT} --name ${CONTAINER_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
        '''
      }
    }

    stage('Verify App') {
      steps {
        echo '🔍 Verifying app is running...'
        sh '''
          echo "Waiting for app to start..."
          sleep 5

          # Get the container's internal IP
          CONTAINER_IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ${CONTAINER_NAME})
          echo "Container IP: $CONTAINER_IP"

          echo "Testing container response..."
          if curl -s http://$CONTAINER_IP:${APP_PORT} | grep -q "Hello"; then
            echo "✅ App is up and responding!"
            exit 0
          else
            echo "❌ App failed to respond on internal IP."
            echo "Retrying via localhost..."
            if curl -s http://localhost:5001 | grep -q "Hello"; then
              echo "✅ App is accessible via localhost!"
              exit 0
            else
              echo "❌ App is not accessible on any interface."
              exit 1
            fi
          fi
        '''
      }
    }
  }

  post {
    success {
      echo "🎉 Jenkins Pipeline completed successfully!"
    }
    failure {
      echo "💥 Jenkins Pipeline failed. Please check logs."
    }
  }
}
