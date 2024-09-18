pipeline {
    agent any

    environment {
        region = "us-east-1"
        docker_repo_uri = "905418229977.dkr.ecr.us-east-1.amazonaws.com/simple-html-app"
        cluster = "simple-html-cluster"
        ecs_service_name = "simple-html-service"
    }

    stages {
        stage('Install Dependencies') {
            steps {
                // ECR login
                sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${docker_repo_uri}"
            }
        }

        stage('Pre-Build') {
            steps {
                script {
                    // Build Docker image
                    sh 'docker build -t sample-app .'
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Tag the Docker image with 'latest'
                    sh "docker tag sample-app:latest ${docker_repo_uri}:latest"
                    
                    // Push the Docker image with the 'latest' tag
                    sh "docker push ${docker_repo_uri}:latest"
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    // Update ECS service to force a new deployment
                    sh "aws ecs update-service --cluster ${cluster} --service ${ecs_service_name} --force-new-deployment"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}
