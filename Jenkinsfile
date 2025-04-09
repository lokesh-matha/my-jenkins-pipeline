pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'lokeshmatha/my-app-image'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_CREDENTIALS = 'docker-hub'
    }

    stages {
        stage('Verify Files') {
            steps {
                script {
                    // Verify critical files exist
                    bat '''
                    echo "Checking required files..."
                    dir /b
                    if not exist requirements.txt (
                        echo "ERROR: requirements.txt missing" && exit 1
                    )
                    if not exist app.py (
                        echo "ERROR: app.py missing" && exit 1
                    )
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        // Build with detailed logging
                        bat '''
                        echo "Building Docker image..."
                        docker build --no-cache -t %DOCKER_IMAGE%:%BUILD_ID% . 2>&1 | tee docker-build.log
                        type docker-build.log
                        '''
                    } catch (Exception e) {
                        echo "Docker build failed: ${e.getMessage()}"
                        bat 'type docker-build.log || echo "No build log found"'
                        error("Build failed")
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    bat '''
                    echo "Running container for tests..."
                    docker run --rm %DOCKER_IMAGE%:%BUILD_ID% python -m pytest || (
                        echo "Tests failed" && exit 1
                    )
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: env.DOCKER_CREDENTIALS,
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        bat '''
                        echo "Logging in to Docker Hub..."
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin

                        echo "Tagging and pushing image..."
                        docker tag %DOCKER_IMAGE%:%BUILD_ID% %DOCKER_IMAGE%:latest
                        docker push %DOCKER_IMAGE%:%BUILD_ID%
                        docker push %DOCKER_IMAGE%:latest
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleanup..."
            script {
                bat '''
                docker system prune -f || echo "Cleanup failed"
                del docker-build.log 2>nul || echo "No log file to remove"
                '''
            }
        }
    }
}
