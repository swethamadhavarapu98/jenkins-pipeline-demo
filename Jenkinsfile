pipeline {
    agent any

    environment {
        IMAGE_NAME = 'swethamadhavarapu/jenkins-pipeline-demo'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Code checked out from GitHub'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t jenkins-pipeline-demo:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: '50e8f804-9dde-499d-acf2-036fde0e01c5',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag jenkins-pipeline-demo:latest $IMAGE_NAME:$IMAGE_TAG
                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Run Container Test') {
            steps {
                sh 'docker rm -f jenkins-pipeline-demo || true'
                sh 'docker run -d -p 8082:80 --name jenkins-pipeline-demo $IMAGE_NAME:$IMAGE_TAG'
                sh 'curl -I http://localhost:8082'
            }
        }
    }

    post {
        always {
            sh 'docker rm -f jenkins-pipeline-demo || true'
        }
    }
}