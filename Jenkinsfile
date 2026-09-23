pipeline {

    agent any

    environment {
        IMAGE_NAME = 'jaganathbkavin/nexus-calculator'
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
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
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

                        $env:DOCKER_PASSWORD | docker login `
                            --username $env:DOCKER_USERNAME `
                            --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            Write-Error "Docker login failed"
                            exit 1
                        }

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

                    docker ps
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
