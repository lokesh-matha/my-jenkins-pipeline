pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'lokeshmatha/my-app-image'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                node {
                    git branch: 'main', url: 'https://github.com/lokesh-matha/my-jenkins-pipeline.git'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image("${DOCKER_IMAGE}:${env.BUILD_ID}").inside {
                        bat 'python -m pytest' // Replace with your test commands
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    bat 'docker stop my-app-container || echo "Container not running"'
                    bat 'docker rm my-app-container || echo "Container not found"'

                    docker.image("${DOCKER_IMAGE}:${env.BUILD_ID}").run(
                        "--name my-app-container -p 5000:5000 -d"
                    )

                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                        docker.image("${DOCKER_IMAGE}:${env.BUILD_ID}").push()
                    }
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
