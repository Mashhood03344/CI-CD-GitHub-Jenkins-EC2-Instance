pipeline {
    agent any

    environment {
        region = "us-east-1"
        docker_repo_uri = "905418229977.dkr.ecr.us-east-1.amazonaws.com/simple-html-app"
        ec2_instance_id = "i-07bc8cba9d91653d9" // Replace with your EC2 instance ID
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

        stage('Deploy to EC2') {
            steps {
                script {
                    def ssmCommand = sh(script: """
                        aws ssm send-command \
                            --region ${region} \
                            --instance-ids ${ec2_instance_id} \
                            --document-name "AWS-RunShellScript" \
                            --comment "Deploy Docker container" \
                            --parameters 'commands=["docker pull ${docker_repo_uri}:latest", "docker stop sample-app || true", "docker rm sample-app || true", "docker run -d -p 80:80 --name sample-app ${docker_repo_uri}:latest"]' \
                            --query "Command.CommandId" --output text
                    """, returnStdout: true).trim()

                    // Poll the SSM output for completion and print results
                    sh """
                        aws ssm get-command-invocation \
                            --command-id ${ssmCommand} \
                            --instance-id ${ec2_instance_id} \
                            --region ${region}
                    """
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

