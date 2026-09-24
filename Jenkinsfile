pipeline {

    agent any

    environment {
        IMAGE_NAME = 'jaganathbkvin/nexus-calculator'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                powershell '''
                    if (!(Test-Path index.html)) {
                        throw "index.html not found"
                    }

                    if (!(Test-Path Dockerfile)) {
                        throw "Dockerfile not found"
                    }

                    Write-Host "Files verified successfully"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                powershell """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

                    if (`$LASTEXITCODE -ne 0) {
                        throw "Docker build failed"
                    }

                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest

                    if (`$LASTEXITCODE -ne 0) {
                        throw "Docker tag failed"
                    }

                    Write-Host "Docker image built successfully"
                """
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    powershell '''
                        Write-Host "Logging into Docker Hub..."

                        docker login -u $env:DOCKER_USERNAME -p $env:DOCKER_PASSWORD

                        Write-Host "Docker login command completed"
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                powershell """
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}

                    if (`$LASTEXITCODE -ne 0) {
                        throw "Docker image push failed"
                    }

                    docker push ${IMAGE_NAME}:latest

                    if (`$LASTEXITCODE -ne 0) {
                        throw "Docker latest image push failed"
                    }

                    Write-Host "Images pushed successfully"
                """
            }
        }

        stage('Deploy') {
            steps {
                powershell '''
                    Write-Host "Stopping old container..."

                    docker rm -f nexus-calculator 2>$null

                    Write-Host "Starting new container..."

                    docker run -d `
                        --name nexus-calculator `
                        -p 8085:80 `
                        jaganathbkvin/nexus-calculator:latest

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker deployment failed"
                    }

                    Write-Host "Application deployed successfully"

                    docker ps --filter name=nexus-calculator
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'BUILD SUCCESSFUL'
            echo 'NEXUS CALCULATOR DEPLOYED'
            echo 'URL: http://localhost:8085'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'BUILD FAILED'
            echo 'Check Jenkins Console Output'
            echo '======================================'
        }
    }
}