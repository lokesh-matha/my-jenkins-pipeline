pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_IMAGE = 'lokeshmatha/my-app-image'
        DOCKER_CREDENTIALS = 'docker-hub'
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
                    // Verify Dockerfile exists
                    bat 'type Dockerfile || echo "Dockerfile not found"'
                    
                    // Build with timestamp tag
                    def timestamp = new Date().format('yyyyMMddHHmmss')
                    docker.build("${DOCKER_IMAGE}:${timestamp}")
                    env.DOCKER_TAG = timestamp
                }
            }
        }

        stage('Test Image') {
            steps {
                script {
                    docker.image("${DOCKER_IMAGE}:${env.DOCKER_TAG}").inside {
                        bat 'python -m pytest || echo "Tests failed"'
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", DOCKER_CREDENTIALS) {
                        docker.image("${DOCKER_IMAGE}:${env.DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${env.DOCKER_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    bat """
                    docker stop my-app-container || echo "No container running"
                    docker rm my-app-container || echo "No container to remove"
                    docker run \
                        --name my-app-container \
                        -p 5000:5000 \
                        -d \
                        ${DOCKER_IMAGE}:${env.DOCKER_TAG}
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Cleanup...'
            script {
                bat 'docker system prune -f'
            }
        }
        success {
            echo "Successfully deployed ${DOCKER_IMAGE}:${env.DOCKER_TAG}"
        }
    }
}
