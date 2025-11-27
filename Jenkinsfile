pipeline {
    agent any

    environment {
        DEPLOY_USER = "johnchuks"
        DEPLOY_HOST = "127.0.1.1"
        DEPLOY_PATH = "/var/www/html"
        BACKUP_PATH = "/var/www/backups"
        REPO_BRANCH = "developer"
        REPO_URL = "https://github.com/JohnehChuks/DevOps.git"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: "${REPO_BRANCH}",
                    url: "${REPO_URL}",
                    credentialsId: 'github-creds'
            }
        }

        stage('Build/Prepare') {
            steps {
                echo "Preparing deployment..."
                sh '''
                    echo "Current workspace: $(pwd)"
                    ls -la
                '''
            }
        }

        stage('Backup Existing Deployment') {
            steps {
                echo "Backing up current deployment if exists..."
                withCredentials([sshUserPrivateKey(credentialsId: 'apache-prod-server', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        if ssh -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "[ -d ${DEPLOY_PATH} ]"; then
                            TIMESTAMP=$(date +%Y%m%d%H%M%S)
                            ssh -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "mkdir -p ${BACKUP_PATH} && cp -r ${DEPLOY_PATH} ${BACKUP_PATH}/backup_${TIMESTAMP}"
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
                echo "Deploying files to ${DEPLOY_USER}@${DEPLOY_HOST}"
                withCredentials([sshUserPrivateKey(credentialsId: 'apache-prod-server', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        rsync -avz -e "ssh -i $SSH_KEY" ./ ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}
                        echo "Deployment finished"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
