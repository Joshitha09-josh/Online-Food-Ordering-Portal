pipeline {
    agent any

    environment {
        DOCKER = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
        KUBE_CONFIG = 'C:\\ProgramData\\Jenkins\\.kube\\config'
        IMAGE_NAME = 'balaji00002/online-food-portal'
        IMAGE_TAG = 'v1'
        DEPLOYMENT_NAME = 'online-food-portal'
        SERVICE_NAME = 'online-food-portal-service'
    }

    stages {
        stage('Clone Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"%DOCKER%" build -t %IMAGE_NAME%:%IMAGE_TAG% .'
            }
        }

        stage('Test Docker Image') {
            steps {
                bat '"%DOCKER%" image inspect %IMAGE_NAME%:%IMAGE_TAG%'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    powershell '''
                        $ErrorActionPreference = "Stop"
                        $env:DOCKER_PASSWORD | & $env:DOCKER login -u $env:DOCKER_USERNAME --password-stdin
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                        & $env:DOCKER push "$env:IMAGE_NAME`:$env:IMAGE_TAG"
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                        & $env:DOCKER logout
                    '''
                }
            }
        }

        stage('Check Kubernetes') {
            steps {
                bat '"%KUBECTL%" --kubeconfig="%KUBE_CONFIG%" get nodes'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '"%KUBECTL%" --kubeconfig="%KUBE_CONFIG%" apply -f deployment.yaml'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat '"%KUBECTL%" --kubeconfig="%KUBE_CONFIG%" rollout status deployment/%DEPLOYMENT_NAME% --timeout=120s'
                bat '"%KUBECTL%" --kubeconfig="%KUBE_CONFIG%" get deployment'
                bat '"%KUBECTL%" --kubeconfig="%KUBE_CONFIG%" get pods'
                bat '"%KUBECTL%" --kubeconfig="%KUBE_CONFIG%" get services'
            }
        }
    }

    post {
        success {
            echo 'Online Food Ordering Portal pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check Console Output for the first failed stage.'
        }
        always {
            echo 'Pipeline execution finished.'
        }
    }
}