
pipeline {
    agent any

    environment {
        DOCKER = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
        KUBECTL = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe'
        IMAGE = 'balaji00002/online-food-portal:v1'
    }

    stages {
        stage('Clone Code') {
            steps {
                echo 'Repository checkout completed by Jenkins SCM configuration'
                bat 'dir'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"%DOCKER%" build -t %IMAGE% .'
            }
        }

        stage('Test Docker Image') {
            steps {
                bat '"%DOCKER%" image inspect %IMAGE%'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    bat '''
                        echo %DOCKER_TOKEN% | "%DOCKER%" login -u %DOCKER_USER% --password-stdin
                        if errorlevel 1 exit /b 1
                        "%DOCKER%" push %IMAGE%
                        if errorlevel 1 exit /b 1
                        "%DOCKER%" logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
    steps {
        bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" apply --validate=false -f deployment.yaml'
    }
}

        stage('Verify Deployment') {
            steps {
                bat '"%KUBECTL%" rollout status deployment/online-food-portal --timeout=120s'
                bat '"%KUBECTL%" get deployment online-food-portal'
                bat '"%KUBECTL%" get pods'
                bat '"%KUBECTL%" get service online-food-portal-service'
            }
        }
    }

    post {
        success {
            echo 'Online Food Ordering pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the first failed stage in Console Output.'
        }
    }
}
