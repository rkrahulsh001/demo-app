pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('cb52b7e8-c9c5-480e-aa7b-649afd815b01')
        IMAGE_NAME = 'rahuls001/demo-app-rahul'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${env.BUILD_ID}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker_hub_creds') {
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    docker container stop demo-app || true
                    docker container rm demo-app || true
                    docker run -d -p 8080:8080 --name demo-app rahuls001/demo-app:latest
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed. Please check the logs."
        }
    }
}
