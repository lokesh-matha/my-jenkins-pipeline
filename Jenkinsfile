pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'lokeshmatha/my-app-image'
        DOCKER_CREDENTIALS = 'docker-hub'
        IMAGE_TAG = "${env.BUILD_ID}-${new Date().format('yyyyMMddHHmm')}" 
    }

    stages {
        stage('Clean Previous') {
            steps {
                script {
                    bat '''
                    echo Cleaning previous containers...
                    docker stop my-app || echo "No container running"
                    docker rm my-app || echo "No container to remove"
                    
                    echo Pruning unused images...
                    docker image prune -f
                    '''
                }
            }
        }

        stage('Build & Tag') {
            steps {
                script {
                    bat """
                    echo Building new image...
                    docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% -t %DOCKER_IMAGE%:latest .
                    """
                }
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKER_CREDENTIALS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    echo Pushing versioned image...
                    docker push %DOCKER_IMAGE%:%IMAGE_TAG%
                    echo Pushing latest...
                    docker push %DOCKER_IMAGE%:latest
                    """
                }
            }
        }

        stage('Deploy & Run') {
            steps {
                script {
                    bat """
                    echo Running Flask container...
                    docker run -d --name my-app -p 5000:5000 %DOCKER_IMAGE%:latest
                    """
                }
            }
        }
    }

    post {
        always {
            script {
                bat '''
                echo Final cleanup...
                docker system prune -f
                '''
            }
        }
    }
}
