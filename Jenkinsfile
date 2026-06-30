stage('Checkout') {
    ...
}

stage('Build Docker Image') {
    ...
}

stage('Push to Docker Hub') {
    ...
}

stage('Deploy to Minikube') {
    steps {
        sh '''
            helm upgrade --install sampleapp ./helm/sampleapp \
              --set image.repository=$IMAGE_NAME \
              --set image.tag=$IMAGE_TAG

            kubectl rollout status deployment/jenkins-pipeline-demo --timeout=120s
            kubectl get pods
            kubectl get svc
        '''
    }
}

stage('Run Container Test') {
    ...
}