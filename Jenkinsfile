pipeline {
    agent any

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

        stage('Run Container Test') {
            steps {
                sh 'docker rm -f jenkins-pipeline-demo || true'
                sh 'docker run -d -p 8082:80 --name jenkins-pipeline-demo jenkins-pipeline-demo:latest'
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