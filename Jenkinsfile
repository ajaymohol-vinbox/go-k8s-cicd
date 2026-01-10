pipeline {
    agent any

    environment {
        // Docker
        DOCKER_IMAGE = 'ajaymoholvinbox/go-k8s-app'
        DOCKER_TAG   = 'latest'

        // Kubernetes
        KUBE_DEPLOYMENT = 'go-app'
        KUBE_CONTAINER  = 'go-app'
        KUBE_NAMESPACE  = 'default'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Go App') {
            steps {
                sh '''
                  echo "Building Go application..."
                  go mod download
                  go build -o app
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  echo "Building Docker image..."
                  docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    withDockerRegistry(
                        credentialsId: 'dockerhub-credentials',
                        url: 'https://index.docker.io/v1/'
                    ) {
                        sh '''
                          echo "Pushing Docker image to Docker Hub..."
                          docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                  echo "Deploying to Kubernetes..."
                  kubectl set image deployment/${KUBE_DEPLOYMENT} \
                  ${KUBE_CONTAINER}=${DOCKER_IMAGE}:${DOCKER_TAG} \
                  --namespace=${KUBE_NAMESPACE}

                  kubectl rollout status deployment/${KUBE_DEPLOYMENT} \
                  --namespace=${KUBE_NAMESPACE}
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 CI/CD Pipeline completed successfully!"
        }
        failure {
            echo "❌ CI/CD Pipeline failed. Check logs."
        }
    }
}
