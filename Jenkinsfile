```groovy
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
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest

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

                        $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin

                        Write-Host "Docker login successful"
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                powershell """
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest

                    Write-Host "Images pushed successfully"
                """
            }
        }

        stage('Deploy') {
            steps {
                powershell '''
                    docker rm -f nexus-calculator 2>$null

                    docker run -d `
                        --name nexus-calculator `
                        -p 8085:80 `
                        jaganathbkavin/nexus-calculator:latest

                    Write-Host "Application deployed successfully"

                    docker ps --filter name=nexus-calculator
                '''
            }
        }
    }

    post {

        success {
            echo 'BUILD SUCCESSFUL'
            echo 'NEXUS CALCULATOR DEPLOYED'
            echo 'URL: http://localhost:8085'
        }

        failure {
            echo 'BUILD FAILED'
        }
    }
}
```
