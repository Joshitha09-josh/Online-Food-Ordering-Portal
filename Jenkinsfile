```groovy
pipeline {
    agent any

    environment {
        DOCKER = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
        KUBE_CONFIG = 'C:\\ProgramData\\Jenkins\\.kube\\config'

        IMAGE_NAME = 'balaji00002/online-food-portal'
        IMAGE_TAG = 'v1'

        DEPLOYMENT_NAME = 'online-food-portal'
        CONTAINER_NAME = 'online-food-portal'
        SERVICE_NAME = 'online-food-portal-service'
    }

    stages {

        stage('Clone Code') {
            steps {
                echo 'Cloning Online Food Ordering Portal from GitHub'
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
                echo 'Docker image build test passed'
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
                    bat '''
                        "%DOCKER%" login -u "%DOCKER_USERNAME%" --password-stdin < nul
                    '''
                    powershell '''
                        $env:DOCKER_PASSWORD | & $env:DOCKER "login" "-u" $env:DOCKER_USERNAME "--password-stdin"
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                        & $env:DOCKER "push" "$env:IMAGE_NAME" + ":" + "$env:IMAGE_TAG"
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                        & $env:DOCKER "logout"
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

        stage('Check Application Endpoint') {
            steps {
                bat '"%KUBECTL%" --kubeconfig="%KUBE_CONFIG%" get service %SERVICE_NAME%'
                echo 'Open http://localhost:30084 to verify the application, provided the Service uses NodePort 30084.'
            }
        }
    }

    post {
        success {
            echo 'Online Food Ordering Portal pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Checking whether the deployment needs rollback.'
            bat '''
                "%KUBECTL%" --kubeconfig="%KUBE_CONFIG%" rollout undo deployment/%DEPLOYMENT_NAME%
            '''
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
```