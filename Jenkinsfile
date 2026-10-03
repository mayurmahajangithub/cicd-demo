pipeline {
    agent any

    environment {
        IMAGE = "cicd-demo"
        TAG   = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Test') {
            steps {
                sh '''
                    python3 -m venv .venv
                    . .venv/bin/activate
                    pip install -q -r requirements.txt -r requirements-dev.txt
                    pytest -q
                '''
            }
        }

        stage('Build image') {
            steps {
                sh 'sudo nerdctl build -t ${IMAGE}:${TAG} --build-arg APP_VERSION=${TAG} .'
            }
        }

        stage('Load image into Kubernetes') {
            steps {
                // Single-node cluster: import the image into containerd's k8s.io namespace
                sh 'sudo nerdctl save ${IMAGE}:${TAG} | sudo ctr -n k8s.io images import -'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    kubectl apply -f k8s/service.yaml
                    sed "s|${IMAGE}:latest|${IMAGE}:${TAG}|" k8s/deployment.yaml | kubectl apply -f -
                    kubectl rollout status deployment/cicd-demo --timeout=90s
                '''
            }
        }
    }

    post {
        success { echo "Deployed ${IMAGE}:${TAG}. Visit http://<VM-IP>:30080" }
        failure { echo "Pipeline failed. Check the stage logs above." }
    }
}
