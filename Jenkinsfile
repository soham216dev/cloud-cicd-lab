
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
                bat '''
                    "%PYTHON_EXE%" -m py_compile app.py
                    if errorlevel 1 exit /b 1

                    "%PYTHON_EXE%" -c "import ast; ast.parse(open('app.py', encoding='utf-8').read()); print('Python syntax test passed')"
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                    "%DOCKER_EXE%" build -t "%DOCKER_IMAGE%" .
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Diagnose Docker Hub Credential') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                        @echo off
                        powershell.exe -NoProfile -NonInteractive -ExecutionPolicy Bypass -Command "$ErrorActionPreference='Stop'; Write-Output ('Docker username correct: ' + ($env:DOCKER_USER -eq 'sohamccd')); Write-Output ('Token populated: ' + (-not [string]::IsNullOrEmpty($env:DOCKER_PASS))); Write-Output ('Token has surrounding whitespace: ' + ($env:DOCKER_PASS -ne $env:DOCKER_PASS.Trim())); Write-Output ('Docker executable exists: ' + (Test-Path $env:DOCKER_EXE))"
                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                        @echo off
                        setlocal DisableDelayedExpansion

                        powershell.exe -NoProfile -NonInteractive -ExecutionPolicy Bypass -Command "$ErrorActionPreference='Stop'; $env:DOCKER_PASS | & $env:DOCKER_EXE login --username $env:DOCKER_USER --password-stdin; exit $LASTEXITCODE"
                        if errorlevel 1 exit /b 1

                        "%DOCKER_EXE%" push "%DOCKER_IMAGE%"
                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                    set "KUBECONFIG=%KUBECONFIG%"
                    "%KUBECTL_EXE%" apply -f k8s\\deployment.yaml
                    if errorlevel 1 exit /b 1
                '''

                bat '''
                    set "KUBECONFIG=%KUBECONFIG%"
                    "%KUBECTL_EXE%" apply -f k8s\\service.yaml
                    if errorlevel 1 exit /b 1
                '''

                bat '''
                    set "KUBECONFIG=%KUBECONFIG%"
                    "%KUBECTL_EXE%" rollout restart deployment/cloud-cicd-deployment
                    if errorlevel 1 exit /b 1
                '''

                bat '''
                    set "KUBECONFIG=%KUBECONFIG%"
                    "%KUBECTL_EXE%" rollout status deployment/cloud-cicd-deployment --timeout=180s
                    if errorlevel 1 exit /b 1
                '''

                bat '''
                    set "KUBECONFIG=%KUBECONFIG%"
                    "%KUBECTL_EXE%" get deployments
                    if errorlevel 1 exit /b 1

                    "%KUBECTL_EXE%" get pods
                    if errorlevel 1 exit /b 1

                    "%KUBECTL_EXE%" get services
                    if errorlevel 1 exit /b 1
                '''
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
