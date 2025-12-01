pipeline {
  agent any

  environment {
    DOCKER_REGISTRY = "docker.io"
    IMAGE_NAME = "vaishnavi873/my-app"
    PYTHON_VENV = "venv"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Prepare Python Virtualenv & Install Python deps') {
      steps {
        // ensure Python is available on the agent PATH
        bat 'python --version'
        bat 'pip --version'

        // Create venv (if not exists) and activate + install
        // On Windows, activation is done with %PYTHON_VENV%\\Scripts\\activate
        bat """
          if not exist %PYTHON_VENV% python -m venv %PYTHON_VENV%
          call %PYTHON_VENV%\\Scripts\\activate
          if exist requirements.txt (
            pip install --upgrade pip
            pip install -r requirements.txt
          ) else (
            echo "No requirements.txt found"
          )
        """
      }
    }

    stage('Optional: Run JS package manager (front-end)') {
      steps {
        // If you have a packageManager.js and node deps for a front-end, run it.
        // This stage will be skipped if packageManager.js doesn't exist.
        bat """
          if exist packageManager.js (
            echo Running custom JS package manager...
            node -v || ( echo Node not found; exit 1 )
            npm -v
            node packageManager.js
          ) else (
            echo "No packageManager.js — skipping JS step"
          )
        """
      }
    }

    stage('Run Tests') {
      steps {
        // Run Python tests with pytest if present; don't fail pipeline if you prefer:
        bat """
          call %PYTHON_VENV%\\Scripts\\activate
          if exist pytest.ini (
            pytest -q
          ) else if exist tests (
            pytest -q || exit 0
          ) else (
            echo "No tests found, skipping pytest"
          )
        """
      }
    }

    stage('Build Docker Image') {
      steps {
        // Build and tag image
        bat "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest ."
      }
    }

    stage('Login & Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USR', passwordVariable: 'DOCKERHUB_PSW')]) {
          bat """
            echo %DOCKERHUB_PSW% | docker login -u %DOCKERHUB_USR% --password-stdin
            docker push ${IMAGE_NAME}:${BUILD_NUMBER}
            docker push ${IMAGE_NAME}:latest
          """
        }
      }
    }

    stage('Cleanup') {
      steps {
        bat "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || exit 0"
        bat "docker rmi ${IMAGE_NAME}:latest || exit 0"
      }
    }
  }

  post {
    always {
      echo "Pipeline finished. Build: ${env.BUILD_NUMBER}"
    }
    failure {
      echo "Pipeline failed for build ${env.BUILD_NUMBER}"
    }
  }
}
