pipeline {
    agent any

    environment {
        REGISTRY = '192.168.1.86:5000'
        IMAGE    = 'myapp'                       // change to your app name
        TAG      = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE:$TAG -t $REGISTRY/$IMAGE:latest .'
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $REGISTRY/$IMAGE:$TAG'
                sh 'docker push $REGISTRY/$IMAGE:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl set image deployment/$IMAGE $IMAGE=$REGISTRY/$IMAGE:$TAG'
                sh 'kubectl rollout status deployment/$IMAGE --timeout=120s'
            }
        }
    }

    post {
        always {
            sh 'docker rmi $REGISTRY/$IMAGE:$TAG $REGISTRY/$IMAGE:latest || true'
        }
    }
}
