pipeline {
    agent any

    environment {
        region = "us-east-1"
        docker_repo_uri = "905418229977.dkr.ecr.us-east-1.amazonaws.com/simple-html-app"
        ec2_instance_id = "i-07bc8cba9d91653d9" 
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
                    // Authenticate EC2 instance to ECR before pulling the image
                    def ssmCommand = sh(script: """
                        aws ssm send-command \
                            --region ${region} \
                            --instance-ids ${ec2_instance_id} \
                            --document-name "AWS-RunShellScript" \
                            --comment "Authenticate and Deploy Docker container" \
                            --parameters 'commands=["aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${docker_repo_uri}", "docker pull ${docker_repo_uri}:latest", "docker stop sample-app || true", "docker rm sample-app || true", "docker run -d -p 80:80 --name sample-app ${docker_repo_uri}:latest"]' \
                            --query "Command.CommandId" --output text
                    """, returnStdout: true).trim()

                    // Poll the SSM command until it's finished
                    def status = "InProgress"
                    while (status == "InProgress") {
                        sleep(time: 10, unit: 'SECONDS')
                        status = sh(script: """
                            aws ssm get-command-invocation \
                                --command-id ${ssmCommand} \
                                --instance-id ${ec2_instance_id} \
                                --region ${region} \
                                --query "Status" --output text
                        """, returnStdout: true).trim()
                        echo "SSM command status: ${status}"
                    }

                    // Check final status
                    if (status != "Success") {
                        error "SSM command failed with status: ${status}"
                    }
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

