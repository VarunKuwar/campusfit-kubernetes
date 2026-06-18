pipeline {
    agent any

    environment {
        IMAGE_NAME = "varunkuwar2005/campusfit"
        BUILD_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Getting code from GitHub...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                sh """
                docker build \
                -t ${IMAGE_NAME}:${BUILD_TAG} \
                -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Push Docker Image') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push ${IMAGE_NAME}:${BUILD_TAG}
                    docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {

                echo 'Deploying to K8s...'

                sh """

                kubectl apply -f k8s/

                kubectl rollout status deployment/campusfit-app
                """
            }
        }

        stage('Verify Deployment') {
            steps {

                echo 'Checking pods...'

                sh '''
                kubectl get pods

                kubectl get svc
                '''
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }

        always {
            sh 'docker image prune -f'
        }
    }
}
