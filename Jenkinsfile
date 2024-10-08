pipeline {
    agent any
    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub-repo'
        IMAGE_NAME = 'chrisgospel12/portfolio'
        REPO_URL = 'https://github.com/kris575/Obeya_profile-Dev.git'
        EC2_INSTANCE_IP = '52.12.197.76' // Updated IP address
    }
    stages {
        stage('Git Checkout') {
            steps {
                // Clone the specified repository
                git url: REPO_URL, branch: 'main'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh 'docker build -t ${IMAGE_NAME} -f Dockerfile .'
                }
            }
        }
        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-repo', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                    sh 'docker push ${IMAGE_NAME}'
                }
            }
        }
        stage('Deploy with Docker Compose to EC2') {
            steps {
                sshagent(credentials: ['EC2-SSH-Key']) {
                    // Copy the compose file to the EC2 instance
                    sh 'scp -o StrictHostKeyChecking=no compose.yml ubuntu@${EC2_INSTANCE_IP}:/home/ubuntu/compose.yml'
                    
                    // Deploy using docker-compose with sudo
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@${EC2_INSTANCE_IP} "sudo docker-compose -f /home/ubuntu/compose.yml up -d"'
                }
            }
        }
    }
}
