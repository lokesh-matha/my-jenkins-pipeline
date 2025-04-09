pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'lokeshmatha/my-app-image'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_CREDENTIALS = 'docker-hub'
    }

    stages {
        stage('Initialize Docker') {
            steps {
                script {
                    bat '''
                    echo Checking Docker status...
                    docker ps || (
                        echo "Docker not responding - restarting Docker Desktop"
                        taskkill /IM "Docker Desktop.exe" /F
                        timeout /t 10
                        start "" "C:\\Program Files\\Docker\\Docker\\Docker Desktop.exe"
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

    stage('Build Docker Image') {
        steps {
            script {
                bat '''
                echo Building Docker image...
                docker build -t %DOCKER_IMAGE%:%BUILD_ID% .
                if errorlevel 1 (
                    echo ERROR: Docker build failed
                    exit /b 1
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
                        bat """
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                        docker push %DOCKER_IMAGE%:%BUILD_ID%
                        """
                    }
                }
            }
        }
    }
}
