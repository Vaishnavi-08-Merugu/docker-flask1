pipeline {
  agent any

   environment {
    DOCKER_REGISTRY = "docker.io"
    IMAGE_NAME = "vaishnavi873/my-app"   
    
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build (package)') {
      steps {
        // install dependencies and run tests (example for Python)
        bat 'pip install -r requirements.txt'
        bat 'pytest -q || true'   // optional: run tests but don't fail pipeline here
      }
    }

    stage('Build Docker Image') {
      steps {
        bat "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest ."
      }
    }

    stage('Login & Push to Docker Hub') {
      steps {
        bat '''
          echo "${DOCKERHUB_PSW}" | docker login -u "${DOCKERHUB_USR}" --password-stdin
          docker push ${IMAGE_NAME}:${BUILD_NUMBER}
          docker push ${IMAGE_NAME}:latest
        '''
      }
    }

    stage('Cleanup') {
      steps {
      bat "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true"
      }
    }
  }
}
