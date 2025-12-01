pipeline {
  agent any

  environment {
    IMAGE_NAME = "vaishnavi873/my-app"  // change this
    DOCKERHUB = credentials('dockerhub')    // create this credential in Jenkins
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build (package)') {
      steps {
        // install dependencies and run tests (example for Python)
        sh 'pip install -r requirements.txt'
        sh 'pytest -q || true'   // optional: run tests but don't fail pipeline here
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest ."
      }
    }

    stage('Login & Push to Docker Hub') {
      steps {
        sh '''
          echo "${DOCKERHUB_PSW}" | docker login -u "${DOCKERHUB_USR}" --password-stdin
          docker push ${IMAGE_NAME}:${BUILD_NUMBER}
          docker push ${IMAGE_NAME}:latest
        '''
      }
    }

    stage('Cleanup') {
      steps {
        sh "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true"
      }
    }
  }
}
