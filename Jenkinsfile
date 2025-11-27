pipeline {
    agent any
    environment {
        DEPLOY_USER = 'johnchuks'
        DEPLOY_HOST = '127.0.1.1'
        DEPLOY_PATH = '/var/www/html'
        BACKUP_PATH = '/var/www/backup'
    }
    stages {
        stage('Checkout SCM') {
            steps {
                echo "Checking out repository..."
                git(
                    url: 'https://github.com/JohnehChuks/DevOps.git',
                    branch: 'developer'
                )
            }
        }
        stage('Build/Prepare') {
            steps {
                echo "Preparing deployment..."
                sh '''
                    pwd
                    ls -la
                '''
            }
        }
        stage('Backup Existing Deployment') {
            steps {
                echo "Backing up current deployment if exists..."
                withCredentials([sshUserPrivateKey(credentialsId: 'apache-prod-server', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                    sh '''
                        chmod 600 "$SSH_KEY"
                        if ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} "[ -d ${DEPLOY_PATH} ]"; then
                            TIMESTAMP=$(date +%Y%m%d%H%M%S)
                            ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} "mkdir -p ${BACKUP_PATH} && cp -r ${DEPLOY_PATH} ${BACKUP_PATH}/backup_${TIMESTAMP}"
                            echo "Backup completed"
                        else
                            echo "No existing deployment to backup"
                        fi
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying files to ${DEPLOY_USER}@${DEPLOY_HOST}..."
                withCredentials([sshUserPrivateKey(credentialsId: 'apache-prod-server', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                    sh '''
                        chmod 600 "$SSH_KEY"
                        rsync -avz --exclude='.git' -e "ssh -i $SSH_KEY -o StrictHostKeyChecking=no" ./ ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}
                        echo "Deployment finished"
                    '''
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                echo "Verifying deployment..."
                withCredentials([sshUserPrivateKey(credentialsId: 'apache-prod-server', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        chmod 600 "$SSH_KEY"
                        ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} "ls -la ${DEPLOY_PATH}"
                    '''
                }
            }
        }
    }
    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
