pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'ajaymoholvinbox/go-k8s-app'
        DOCKER_TAG = 'latest'
        KUBE_DEPLOYMENT = 'go-app'
        KUBE_CONTAINER = 'go-app'
        KUBE_NAMESPACE = 'default'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Go App') {
            steps {
                sh 'go mod download'
                sh 'go build -o app'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) {
                    script {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl set image deployment/${KUBE_DEPLOYMENT} ${KUBE_CONTAINER}=${DOCKER_IMAGE}:${DOCKER_TAG} --namespace=${KUBE_NAMESPACE}
                """
            }
        }
    }
}

