pipeline {
  agent any

  environment {
    IMAGE_NAME = "hello-d"
    IMAGE_TAG  = "latest"
    CONTAINER_NAME = "hello-d"
    APP_PORT = "5001"
  }

  stages {
    stage('Checkout') {
      steps {
        echo 'Checking out source code...'
      }
    }

    stage('Build Docker Image') {
      steps {
        echo 'Building Docker image...'
        sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
      }
    }

    stage('Run Container') {
  steps {
    echo 'Running the container...'
    sh '''
      if [ "$(docker ps -q -f name=${CONTAINER_NAME})" ]; then
        docker rm -f ${CONTAINER_NAME} || true
      fi
      docker run -d --network host --name ${CONTAINER_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
    '''
  }
}

    stage('Verify App') {
  steps {
    echo 'Verifying app is running...'
    sh '''
      echo "Waiting for app..."
      for i in {1..10}; do
        if curl -s http://localhost:5000 | grep -q "Hello"; then
          echo "App is up!"
          exit 0
        fi
        sleep 2
      done
      echo "App failed to respond"
      exit 1
    '''
  }
}

  post {
    success {
      echo "Jenkins Pipeline completed successfully."
    }
    failure {
      echo "Jenkins Pipeline failed."
    }
  }
}
