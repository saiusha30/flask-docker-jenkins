pipeline {
    agent any
    environment {
        IMAGE_NAME = 'flask-app'
        TARGET_PATH = '/opt/flask-app/deployments'
        TARGET_HOST = 'user@target-machine'
    }
    stages {
        stage('Build') {
            steps {
                script {
                    sh 'docker build -t ${IMAGE_NAME} .'
                }
            }
        }
        stage('Save Image') {
            steps {
                script {
                    sh 'docker save ${IMAGE_NAME} | gzip > ${IMAGE_NAME}.tar.gz'
                }
            }
        }
        stage('Transfer Image') {
            steps {
                script {
                    sshagent(['your-ssh-credentials-id']) {
                        sh 'scp ${IMAGE_NAME}.tar.gz ${TARGET_HOST}:${TARGET_PATH}'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sshagent(['your-ssh-credentials-id']) {
                        sh 'ssh ${TARGET_HOST} "gunzip -c ${TARGET_PATH}/${IMAGE_NAME}.tar.gz | docker load"'
                        sh 'ssh ${TARGET_HOST} "docker run -d -p 5000:5000 ${IMAGE_NAME}"'
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'docker system prune -af'
        }
    }
}

