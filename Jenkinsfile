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
                    credentialsId: '50ef8f04-9dde-499d-acf2-036fde0e01c5',
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

        stage('Deploy to DEV') {
            steps {
                sh '''
                    helm upgrade --install sampleapp-dev ./helm/sampleapp \
                      --namespace dev \
                      --create-namespace \
                      --set image.repository=$IMAGE_NAME \
                      --set image.tag=$IMAGE_TAG \
                      --set service.nodePort=30081

                    kubectl rollout status deployment/jenkins-pipeline-demo -n dev --timeout=120s
                    kubectl get pods -n dev
                    kubectl get svc -n dev
                '''
            }
        }

        stage('Approval Before PROD') {
            steps {
                input message: 'Approve deployment to PROD?', ok: 'Deploy to PROD'
            }
        }

        stage('Deploy to PROD') {
            steps {
                sh '''
                    helm upgrade --install sampleapp-prod ./helm/sampleapp \
                      --namespace prod \
                      --create-namespace \
                      --set image.repository=$IMAGE_NAME \
                      --set image.tag=$IMAGE_TAG \
                      --set service.nodePort=30082

                    kubectl rollout status deployment/jenkins-pipeline-demo -n prod --timeout=120s
                    kubectl get pods -n prod
                    kubectl get svc -n prod
                '''
            }
        }

        stage('Run Container Test') {
            steps {
                sh 'docker rm -f jenkins-pipeline-demo || true'
                sh 'docker run -d -p 8082:80 --name jenkins-pipeline-demo $IMAGE_NAME:$IMAGE_TAG'
                sh 'sleep 5'
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