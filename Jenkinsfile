pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'lokeshmatha/node-ci-cd-demo'
        DOCKER_TAG = 'latest'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/lokesh-matha/my-jenkins-pipeline'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        stage('Test') {
            steps {
                sh 'echo "Running tests..."'
                // Example: sh 'npm test' or './gradlew test'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                    // Optional: Deploy to a server
                    sh "docker run -d -p 8081:80 ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
    }
}
