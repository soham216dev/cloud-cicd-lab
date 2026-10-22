
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

        stage('Test Docker Hub Login') {
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
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Application tests, Docker build, and Docker Hub login test completed successfully.'
            echo 'This diagnostic pipeline does not push the image or deploy to Kubernetes.'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs above.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
