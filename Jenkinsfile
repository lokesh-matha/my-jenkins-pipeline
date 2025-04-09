pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'lokeshmatha/my-app-image'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").inside {
                        sh 'python -m pytest' // Replace with your test commands
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                script {
                    sh 'docker stop my-app-container || true'
                    sh 'docker rm my-app-container || true'
                    sh "docker run --name my-app-container -p 5000:5000 -d ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished'
        }
        success {
            echo 'Application deployed successfully!'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
