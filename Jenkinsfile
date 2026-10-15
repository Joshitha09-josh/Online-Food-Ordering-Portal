
pipeline {
    agent any

    stages {
        stage('Clone Code') {
            steps {
                echo 'Getting source code from GitHub'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe" build -t balaji00002/online-food-portal:v1 .'
            }
        }

        stage('Test Docker Image') {
            steps {
                bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe" images balaji00002/online-food-portal:v1'
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
                    bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe" login -u "%DOCKER_USERNAME%" -p "%DOCKER_PASSWORD%"'
                    bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe" push balaji00002/online-food-portal:v1'
                    bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe" logout'
                }
            }
        }

        stage('Check Kubernetes') {
            steps {
                bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" config current-context'
                bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" get nodes'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" apply -f deployment.yaml'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" rollout status deployment/online-food-portal'
                bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" get deployments'
                bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" get pods'
                bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\kubectl.exe" get services'
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the first failed stage in Console Output.'
        }
    }
}
