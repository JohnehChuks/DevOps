pipeline {
    agent any

    environment {
        DEPLOY_USER = "johnchuks"  // updated from "ubuntu"
        DEPLOY_HOST = "172.18.245.183"
        DEPLOY_PATH = "/var/www/html"
        BACKUP_PATH = "/var/www/backups"
        SSH_CREDENTIALS = "apache-prod-server"
        REPO_BRANCH = "developer" // change to your branch
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: "${REPO_BRANCH}",
                    url: 'https://github.com/JohnehChuks/DevOps.git', // updated repo URL
                    credentialsId: 'github-creds'
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    echo "Build your project here"
                    # Example: npm install && npm run build
                    # Example: mvn clean package
                '''
            }
        }

        stage('Package Artifact') {
            steps {
                sh '''
                    rm -rf build
                    mkdir -p build
                    cp -r * build/
                '''
                archiveArtifacts artifacts: 'build/**', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                sshagent([SSH_CREDENTIALS]) {
                    sh """
                        echo "Backing up current site..."
                        ssh ${DEPLOY_USER}@${DEPLOY_HOST} '
                            mkdir -p ${BACKUP_PATH}
                            TS=`date +%Y%m%d%H%M%S`
                            sudo tar -czf ${BACKUP_PATH}/html.bak.${TS}.tar.gz -C /var/www html || true
                        '

                        echo "Deploying new version..."
                        rsync -avz --delete build/ ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}

                        echo "Restarting Apache..."
                        ssh ${DEPLOY_USER}@${DEPLOY_HOST} "sudo systemctl restart apache2"

                        echo "Deployment completed!"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed! Attempting rollback..."
            sshagent([SSH_CREDENTIALS]) {
                sh """
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} '
                        PREV=$(ls -1t ${BACKUP_PATH}/html.bak.*.tar.gz | head -n1 || true)
                        if [ -n "$PREV" ]; then
                            echo "Restoring backup $PREV"
                            sudo tar -xzf "$PREV" -C /
                            sudo systemctl restart apache2
                        else
                            echo "No backup available"
                        fi
                    '
                """
            }
        }
        always {
            cleanWs()
        }
    }
}
