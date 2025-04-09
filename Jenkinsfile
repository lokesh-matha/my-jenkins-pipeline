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
                    // Verify critical files exist (Windows compatible)
                    bat '''
                    echo Checking required files...
                    dir
                    if not exist requirements.txt (
                        echo ERROR: requirements.txt missing && exit /b 1
                    )
                    if not exist app.py (
                        echo ERROR: app.py missing && exit /b 1
                    )
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        // Windows-compatible build with logging
                        bat '''
                        echo Building Docker image...
                        docker build --no-cache -t %DOCKER_IMAGE%:%BUILD_ID% . > docker-build.log 2>&1
                        type docker-build.log
                        '''
                        
                        // Verify image was created
                        bat '''
                        docker images | find "%DOCKER_IMAGE%:%BUILD_ID%"
                        if errorlevel 1 (
                            echo ERROR: Image not built successfully && exit /b 1
                        )
                        '''
                    } catch (Exception e) {
                        echo "Docker build failed: ${e.getMessage()}"
                        bat 'if exist docker-build.log (type docker-build.log) else (echo No build log found)'
                        error("Build failed")
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    bat '''
                    echo Running container for tests...
                    docker run --rm %DOCKER_IMAGE%:%BUILD_ID% python -m pytest
                    if errorlevel 1 (
                        echo ERROR: Tests failed && exit /b 1
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
                        echo Logging in to Docker Hub...
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin

                        echo Tagging and pushing image...
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
                docker system prune -f
                if exist docker-build.log del docker-build.log
                '''
            }
        }
    }
}
