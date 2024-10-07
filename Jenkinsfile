pipeline {
    agent any

    environment {
        // Docker Hub credentials
        DOCKER_HUB_CREDENTIALS = 'dockerhub-repo'
        DOCKER_IMAGE = 'chrisgospel12/portfolio:latest'
        REPO_URL = 'https://github.com/kris575/Obeya_profile-Dev.git'
        EC2_INSTANCE_IP = '34.221.161.245' // Your EC2 instance public IP
        SSH_CREDENTIALS_ID = 'ec2-ssh-keypair' // Jenkins credential ID for the EC2 SSH keypair
        EC2_USER = 'ubuntu' // Change to 'ec2-user' if using Amazon Linux
    }

    stages {
        stage('Git Checkout') {
            steps {
                // Pull the code from GitHub
                git branch: 'main', url: REPO_URL
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub and push the image
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                // Use SSH Agent plugin to use credentials securely
                sshagent([SSH_CREDENTIALS_ID]) {
                    // SSH into EC2 instance and run deployment commands
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_INSTANCE_IP} << EOF
                            sudo docker pull ${DOCKER_IMAGE}
                            sudo docker stop yourapp || true
                            sudo docker rm yourapp || true
                            sudo docker run -d --name yourapp -p 80:80 ${DOCKER_IMAGE}
                            exit
                        EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}
