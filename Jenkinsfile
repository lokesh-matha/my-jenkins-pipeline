pipeline {
    agent any

    // Required plugins declaration
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        DOCKER_IMAGE = 'lokeshmatha/my-app-image'
        DOCKER_HOST = 'tcp://localhost:2375'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Clean up any existing containers
                    bat 'docker stop my-app-container || echo "No container to stop"'
                    bat 'docker rm my-app-container || echo "No container to remove"'
                    
                    // Build with resource limits
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}", 
                        "--cpu-quota 100000 --memory 1G .")
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    bat """
                    docker run \
                        --name my-app-container \
                        -p 5000:5000 \
                        --cpus=1 \
                        --memory=1g \
                        -d \
                        ${DOCKER_IMAGE}:${env.BUILD_ID}
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            script {
                bat 'docker system prune -f || echo "Cleanup failed"'
            }
        }
    }
}
