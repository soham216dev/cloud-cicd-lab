
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'sohamccd/cloud-cicd-lab:latest'
        DOCKER_EXE = 'C:\\Users\\soham\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Users\\soham\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'
        KUBECONFIG = 'C:\\Users\\soham\\.kube\\config'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/soham216dev/cloud-cicd-lab.git'
            }
        }

        stage('Test Application') {
            steps {
                bat 'python -m py_compile app.py'
                bat 'python -m pip show flask'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"%DOCKER_EXE%" build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat '''
                        @echo off
                        echo %DOCKER_PASS%| "%DOCKER_EXE%" login -u "%DOCKER_USER%" --password-stdin
                        if errorlevel 1 exit /b 1
                        "%DOCKER_EXE%" push "%DOCKER_IMAGE%"
                        if errorlevel 1 exit /b 1
                        "%DOCKER_EXE%" logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat 'set KUBECONFIG=%KUBECONFIG% && "%KUBECTL_EXE%" apply -f k8s\\deployment.yaml'
                bat 'set KUBECONFIG=%KUBECONFIG% && "%KUBECTL_EXE%" apply -f k8s\\service.yaml'
                bat 'set KUBECONFIG=%KUBECONFIG% && "%KUBECTL_EXE%" set image deployment/cloud-cicd-deployment cloud-cicd-app=%DOCKER_IMAGE%'
                bat 'set KUBECONFIG=%KUBECONFIG% && "%KUBECTL_EXE%" rollout status deployment/cloud-cicd-deployment --timeout=120s'
                bat 'set KUBECONFIG=%KUBECONFIG% && "%KUBECTL_EXE%" get pods'
                bat 'set KUBECONFIG=%KUBECONFIG% && "%KUBECTL_EXE%" get services'
            }
        }
    }
}
