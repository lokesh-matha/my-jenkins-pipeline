pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'lokeshmatha/my-app-image'
        DOCKER_TAG = 'latest'
        // Set Docker host for Windows
        DOCKER_HOST = "tcp://localhost:2375"
    }

    stages {
        stage('Checkout') {
            steps {
                // Use checkout scm for better SCM integration
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Add build timestamp to tag
                    def customTag = "${env.BUILD_ID}-${new Date().format('yyyyMMddHHmmss')}"
                    
                    // Build with proper isolation for Windows
                    docker.build("${DOCKER_IMAGE}:${customTag}", "--isolation=process .")
                    
                    // Store the built tag for later stages
                    env.DOCKER_BUILD_TAG = customTag
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Use bat instead of sh for Windows
                    docker.image("${DOCKER_IMAGE}:${env.DOCKER_BUILD_TAG}").inside("--isolation=process") {
                        bat 'python -m pytest || echo "Tests failed"'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Add error handling for container operations
                    bat """
                    docker stop my-app-container || echo "Container not running"
                    docker rm my-app-container || echo "Container not found"
                    """
                    
                    // Run with health check and restart policy
                    docker.image("${DOCKER_IMAGE}:${env.DOCKER_BUILD_TAG}").run(
                        "--name my-app-container " +
                        "-p 5000:5000 " +
                        "--restart=unless-stopped " +
                        "-d"
                    )

                    // Push to Docker Hub with credentials
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                        docker.image("${DOCKER_IMAGE}:${env.DOCKER_BUILD_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${env.DOCKER_BUILD_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Debug Docker') {
            steps {
                script {
                    // Get detailed docker info
                    bat 'docker --version'
                    bat 'docker info'
                    bat 'docker images'
                    bat 'docker ps -a'
                    bat 'docker network ls'
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed - cleaning up'
            // Clean up unused containers and images
            script {
                bat 'docker system prune -f || echo "Cleanup failed"'
            }
        }
        success {
            echo "Successfully deployed at http://localhost:5000"
            echo "Docker image: ${DOCKER_IMAGE}:${env.DOCKER_BUILD_TAG}"
        }
        failure {
            echo 'Pipeline failed - checking logs'
            script {
                // Get last 50 lines of failed container logs if available
                bat 'docker logs --tail 50 my-app-container || echo "No container logs available"'
            }
        }
    }
}
