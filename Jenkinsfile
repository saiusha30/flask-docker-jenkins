pipeline {
    agent any

    environment {
        // Replace with your actual SSH credentials ID from Jenkins
        SSH_CREDENTIALS_ID = 'e9ccc7b7-6d94-4049-a3e4-5b1cb95c797e'
        EC2_USERNAME = 'ec2-user'
        EC2_HOST = '35.154.193.71'
        GIT_REPO = 'https://github.com/saiusha30/flask-docker-jenkins'
        GIT_BRANCH = 'main'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh 'docker build -t flask-app .'
                }
            }
        }

        stage('Save Docker Image') {
            steps {
                script {
                    // Save the Docker image to a file
                    sh 'docker save flask-app | gzip > flask-app.tar.gz'
                }
            }
        }

        stage('Transfer Docker Image to EC2') {
            steps {
                script {
                    // Use SSH agent to transfer the Docker image
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh '''
                        scp -o StrictHostKeyChecking=no flask-app.tar.gz ${EC2_USERNAME}@${EC2_HOST}:/home/${EC2_USERNAME}/
                        '''
                    }
                }
            }
        }

        stage('Deploy Docker Image on EC2') {
            steps {
                script {
                    // Use SSH to deploy the Docker image on EC2
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh '''
                        ssh -o StrictHostKeyChecking=no ${EC2_USERNAME}@${EC2_HOST} << 'EOF'
                        docker load < /home/${EC2_USERNAME}/flask-app.tar.gz
                        docker run -d -p 5000:5000 flask-app
                        EOF
                        '''
                    }
                }
            }
        }

        stage('Clean up') {
            steps {
                script {
                    // Clean up any unused Docker images and containers
                    sh 'docker system prune -af'
                }
            }
        }
    }
}
