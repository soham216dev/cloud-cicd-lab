
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    environment {
        DOCKER_IMAGE = 'sohamccd/cloud-cicd-lab:latest'
        DOCKER_EXE = 'C:\\Users\\soham\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL_EXE = 'C:\\Users\\soham\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'
        PYTHON_EXE = 'C:\\Users\\soham\\AppData\\Local\\Programs\\Python\\Python313\\python.exe'
        KUBECONFIG = 'C:\\Users\\soham\\.kube\\config'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test Application') {
            steps {
                bat '"%PYTHON_EXE%" -m py_compile app.py'
                bat '"%PYTHON_EXE%" -c "import ast; ast.parse(open(\'app.py\', encoding=\'utf-8\').read()); print(\'Python syntax test passed\')"'
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
                        powershell.exe -NoProfile -NonInteractive -Command "$ErrorActionPreference = 'Stop'; $env:DOCKER_PASS | & $env:DOCKER_EXE login --username $env:DOCKER_USER --password-stdin"
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

                bat 'set KUBECONFIG=%KUBECONFIG% && "%KUBECTL_EXE%" rollout restart deployment/cloud-cicd-deployment'
                bat 'set KUBECONFIG=%KUBECONFIG% && "%KUBECTL_EXE%" rollout status deployment/cloud-cicd-deployment --timeout=180s'

                bat 'set KUBECONFIG=%KUBECONFIG% && "%KUBECTL_EXE%" get deployments'
                bat 'set KUBECONFIG=%KUBECONFIG% && "%KUBECTL_EXE%" get pods'
                bat 'set KUBECONFIG=%KUBECONFIG% && "%KUBECTL_EXE%" get services'
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'CI/CD pipeline failed. Check the stage logs above.'
        }
        always {
            echo 'Pipeline execution finished.'
        }
    }
}
