pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {

                git branch: 'main', url: 'https://github.com/lokesh-matha/my-jenkins-pipeline.git'
                git branch: 'main', url: 'https://github.com/lokesh-matha/my-jenkins-pipeline'
            }
        }

        stage('Build') {
            steps {
                script {
                    docker.build("my-app-image:${env.BUILD_ID}")
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image("my-app-image:${env.BUILD_ID}").inside {
                        bat 'python -m pytest'  // Windows uses bat instead of sh
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    bat 'docker stop my-app-container || echo "Container not running"'
                    bat 'docker rm my-app-container || echo "Container not found"'
                    
                    docker.image("my-app-image:${env.BUILD_ID}").run(
                        "--name my-app-container -p 5000:5000 -d"
                    )
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                    // Optional: Deploy to a server
                    sh "docker run -d -p 8081:80 ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed - cleaning up'
        }
        success {
            echo 'Deployed at http://localhost:5000'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}

