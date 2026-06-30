stage('Push to Docker Hub') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: '50e8f804-9dde-499d-acf2-036fde0e01c5',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh '''
                docker login -u $DOCKER_USER -p $DOCKER_PASS
                docker tag jenkins-pipeline-demo:latest swethamadhavarapu/jenkins-pipeline-demo:latest
                docker push swethamadhavarapu/jenkins-pipeline-demo:latest
            '''
        }
    }
}