pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials-id')
        AWS_CREDENTIALS = credentials('aws-credentials-id')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo/spring-boot-app.git'
            }
        }

        stage('Build') {
            steps {
                sh './gradlew clean build'
            }
        }

        stage('Test') {
            steps {
                sh './gradlew test'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.build('your-dockerhub-username/spring-boot-app:latest')
                    docker.withRegistry('https://registry.hub.docker.com', 'DOCKER_HUB_CREDENTIALS') {
                        docker.image('your-dockerhub-username/spring-boot-app:latest').push()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region your-region | docker login --username AWS --password-stdin your-account-id.dkr.ecr.your-region.amazonaws.com'
                    sh 'docker tag spring-boot-app:latest your-account-id.dkr.ecr.your-region.amazonaws.com/spring-boot-app:latest'
                    sh 'docker push your-account-id.dkr.ecr.your-region.amazonaws.com/spring-boot-app:latest'
                    sh 'aws ecs update-service --cluster your-cluster --service your-service --force-new-deployment'
                }
            }
        }
    }

    post {
        always {
            junit 'build/test-results/test/*.xml'
            archiveArtifacts artifacts: 'build/libs/*.jar', allowEmptyArchive: true
        }
    }
}
