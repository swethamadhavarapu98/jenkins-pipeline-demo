@Library('jenkins-shared-library') _

pipeline {
    agent any

    environment {
        IMAGE_NAME = 'swethamadhavarapu/jenkins-pipeline-demo'
        IMAGE_TAG = 'latest'
        DOCKER_CREDENTIALS_ID = '50ef8f04-9dde-499d-acf2-036fde0e01c5'
    }

    stages {

        stage('Generate Tag') {
            steps {
                echo "Generated Docker image tag: ${IMAGE_TAG}"
            }
        }

        stage('Checkout') {
            steps {
                echo 'Code checked out from GitHub'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dockerBuild(IMAGE_NAME)
            }
        }

        stage('Push to Docker Hub') {
            steps {
                dockerPush(IMAGE_NAME, DOCKER_CREDENTIALS_ID)
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