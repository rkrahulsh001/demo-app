pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "rahuls001/demo-app"
        PATH = "${tool 'docker'}/bin:${env.PATH}"
        EMAIL_RECIPIENT = "rkrahulsh001@gmail.com"
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
                    sh """
                    docker build -t ${DOCKER_IMAGE}:latest .
                    """
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh """
                        docker login -u rahuls001 -p $DOCKER_PASS
                        docker push ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }
        stage('Deploy Application') {
            steps {
                script {
                    sh """
                    docker service rm demo-app || true
                    docker service create --name demo-app --replicas 2 -p 3000:3000 ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
    }

    post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

                def body = """<html>
                                <body>
                                    <div style="border: 4px solid ${bannerColor}; padding: 10px";>
                                        <h2>$(jobName) - Build ${buildNumber}</h2>
                                        <div style="background-color: ${bannerColor}; padding: 10px;">
                                            <h3 style="color: white;"> Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>;
                                        </div>
                                        <p>Check the <a href="%{BUILD_URL}">console output</a></p>
                                    </div>
                                </body>
                            </html>"""
                emailext (
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                    body: body,
                    to: "rkrahulsh001@gmail.com",
                    replyTo: "rkrahulsh001@gmail.com",
                    mimeType: "text/html"
                )
            }
        }
    }
}
