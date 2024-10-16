pipeline {
    agent any

    environment {
        // Define your variables here
        GITHUB_REPO = 'https://github.com/Rithigasri/Capstone-Guvi.git' // Replace with your GitHub repository URL
        DEPLOY_DIR = '/var/www/html' // Apache default document root
        SSH_CREDENTIALS_ID = '9e5eefe4-055a-4706-bcf4-1e8175379236' // Jenkins credentials ID for SSH
        EC2_USER = 'ubuntu' // Replace with your EC2 user if different
        EC2_HOST = '3.143.209.116' // Replace with your EC2 instance public IP
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Clone the GitHub repository
                    echo "Cloning repository from ${GITHUB_REPO}..."
                    git branch: 'main', url: "${GITHUB_REPO}"
                }
            }
        }

        stage('Deploy to Apache') {
            steps {
                script {
                    // Sync files to the Apache server using SSH
                    echo "Deploying to Apache server at ${EC2_HOST}..."
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            # Copy files to the Apache document root
                            scp -r -o StrictHostKeyChecking=no * ${EC2_USER}@${EC2_HOST}:${DEPLOY_DIR}
                        """
                    }
                }
            }
        }

        stage('Start/Restart CloudWatch Agent') {
            steps {
                script {
                    // Ensure CloudWatch Agent is running on the EC2 instance
                    echo "Starting CloudWatch Agent on ${EC2_HOST}..."
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'sudo systemctl restart amazon-cloudwatch-agent'
                        """
                    }
                }
            }
        }

        stage('Verify CloudWatch Agent Status') {
            steps {
                script {
                    // Check the status of CloudWatch Agent
                    echo "Checking CloudWatch Agent status on ${EC2_HOST}..."
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} 'sudo systemctl status amazon-cloudwatch-agent'
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment and CloudWatch monitoring setup was successful!'
        }
        failure {
            echo 'Deployment failed. Check the logs for more information.'
        }
    }
}
