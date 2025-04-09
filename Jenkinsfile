pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'lokeshmatha/my-app-image'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_CREDENTIALS = 'docker-hub-creds'
    }

    stage('Initialize Docker') {
        steps {
            script {
                bat '''
                echo Checking Docker status...
                docker ps || (
                    echo "Docker not responding - restarting Docker Desktop"
                    taskkill /IM "Docker Desktop.exe" /F
                    timeout /t 10
                    start "" "C:\Program Files\Docker\Docker\Docker Desktop.exe"
                    timeout /t 30
                    docker ps || (
                        echo "Failed to start Docker"
                        exit 1
                    )
                )
                '''
            }
        }
    }

        stage('Verify Repository') {
            steps {
                script {
                    // Print workspace structure for debugging
                    bat '''
                    echo Workspace Contents:
                    dir /b
                    '''

                    // Verify all required files exist
                    def requiredFiles = [
                        'app.py',
                        'requirements.txt',
                        'Dockerfile'
                    ]

                    requiredFiles.each { file ->
                        bat """
                        if not exist "${file}" (
                            echo ERROR: Missing required file - ${file}
                            exit /b 1
                        )
                        """
                    }

                    // Verify file contents
                    bat '''
                        echo --- requirements.txt CONTENTS ---
                        type requirements.txt
                        echo --- app.py CONTENTS ---
                        type app.py
                        '''
                }
            }
        }

        stage('Build Docker Image') {
            options {
                timeout(time: 15, unit: 'MINUTES')
            }
            steps {
                script {
                    try {
                        bat """
                        echo Building Docker image with BuildKit...
                        set DOCKER_BUILDKIT=1
                        docker build --progress=plain -t %DOCKER_IMAGE%:%BUILD_ID% . > docker-build.log 2>&1
                        type docker-build.log
                        
                        echo Verifying image was created...
                        docker images | find "%DOCKER_IMAGE%:%BUILD_ID%"
                        if errorlevel 1 (
                            echo ERROR: Docker image not created
                            exit /b 1
                        )
                        """
                    } catch (Exception e) {
                        echo "Docker build failed: ${e.getMessage()}"
                        bat 'if exist docker-build.log (type docker-build.log) else (echo No build log found)'
                        error("Build stage failed")
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    bat """
                    echo Running tests in container...
                    docker run --rm %DOCKER_IMAGE%:%BUILD_ID% python -m pytest
                    if errorlevel 1 (
                        echo ERROR: Tests failed
                        exit /b 1
                    )
                    """
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
                        bat """
                        echo Logging in to Docker Hub...
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin

                        echo Tagging and pushing image...
                        docker tag %DOCKER_IMAGE%:%BUILD_ID% %DOCKER_IMAGE%:latest
                        docker push %DOCKER_IMAGE%:%BUILD_ID%
                        docker push %DOCKER_IMAGE%:latest
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed - performing cleanup"
            script {
                bat '''
                echo Cleaning up containers...
                docker ps -aq | foreach { docker stop $_ }
                docker system prune -f
                
                echo Removing temp files...
                if exist docker-build.log del docker-build.log
                '''
            }
        }
        success {
            echo "Pipeline succeeded! Image available at:"
            echo "${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${BUILD_ID}"
        }
        failure {
            echo "Pipeline failed - checking logs"
            script {
                bat '''
                echo === DEBUG INFO ===
                docker ps -a
                docker images
                docker info
                '''
            }
        }
    }
}
