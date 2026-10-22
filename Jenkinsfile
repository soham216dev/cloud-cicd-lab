pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/soham216dev/cloud-cicd-lab.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t sohamccd/cloud-cicd-lab:latest .'
            }
        }
    }
}