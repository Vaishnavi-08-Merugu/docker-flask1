pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Jenkins stored creds
        IMAGE_NAME = 'vaishnavi873/python-flask-app'       // Replace with your DockerHub repo
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://gitlab.com/Vaishnavi-08-Merugu/docker-flask.git'  // replace with your repo
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Cleanup Local Images') {
            steps {
                sh "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true"
                sh "docker rmi ${IMAGE_NAME}:latest || true"
            }
        }
    }
}
