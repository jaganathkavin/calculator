```groovy
pipeline {

    agent any

    environment {
        IMAGE_NAME = 'jaganathbkvin/nexus-calculator'
        IMAGE_TAG  = "build-${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                powershell '''
                    if (!(Test-Path "index.html")) {
                        Write-Error "index.html not found"
                        exit 1
                    }

                    if (!(Test-Path "Dockerfile")) {
                        Write-Error "Dockerfile not found"
                        exit 1
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

                    Write-Host "Docker images created:"
                    docker images ${IMAGE_NAME}
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

                        $DOCKER_PASSWORD | docker login `
                            --username $DOCKER_USERNAME `
                            --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            throw "Docker login failed"
                        }

                        Write-Host "Docker login successful"
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {

                powershell """
                    Write-Host "Pushing ${IMAGE_NAME}:${IMAGE_TAG}..."
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}

                    if (`$LASTEXITCODE -ne 0) {
                        throw "Build image push failed"
                    }

                    Write-Host "Pushing ${IMAGE_NAME}:latest..."
                    docker push ${IMAGE_NAME}:latest

                    if (`$LASTEXITCODE -ne 0) {
                        throw "Latest image push failed"
                    }

                    Write-Host "Both images pushed successfully."
                """
            }
        }

        stage('Deploy') {
            steps {

                powershell '''
                    Write-Host "Removing old container..."

                    docker rm -f nexus-calculator 2>$null

                    Write-Host "Starting new container..."

                    docker run -d `
                        --name nexus-calculator `
                        -p 8085:80 `
                        jaganathbkvin/nexus-calculator:latest

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker deployment failed"
                    }

                    Write-Host "Container status:"
                    docker ps --filter "name=nexus-calculator"
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'BUILD SUCCESSFUL'
            echo 'NEXUS CALCULATOR DEPLOYED'
            echo '======================================'
            echo 'URL: http://localhost:8085'
        }

        failure {
            echo '======================================'
            echo 'BUILD FAILED'
            echo 'Check Jenkins Console Output'
            echo '======================================'
        }
    }
}
```
