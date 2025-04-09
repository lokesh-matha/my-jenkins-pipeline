pipeline {
    agent any
    stages {
        stage('Test Git') {
            steps {
                bat 'git --version'
            }
        }
    }
}node('slave-node') {

    checkout scm

    stage('Building Image') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
            def customImage = docker.build("avinashmahto/pipeline:latest")
            customImage.push()
        }
    }

    stage('Deploy') {
        sh 'docker stop my-app-container || true'
        sh 'docker rm my-app-container || true'
        sh "docker run --name my-app-container -d -p 5000:5000 avinashmahto/pipeline:latest"
        sh 'docker ps | grep my-app-container || echo "Container not running!"'
    }
}
