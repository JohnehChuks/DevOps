kpipeline {
    agent any

    environment {
        DEPLOY_USER = "johnchuks"
        DEPLOY_HOST = "127.0.1.1"
        DEPLOY_PATH = "/var/www/html"
        BACKUP_PATH = "/var/www/backups"
        SSH_KEY = "/var/lib/jenkins/.ssh/id_rsa_pipeline"
        REPO_BRANCH = "developer"
        REPO_URL = "https://github.com/JohnehChuks/DevOps.git"
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
                    url: "${REPO_URL}",
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
                sh '''
                    echo "Backing up current site..."
                    ssh -i ${SSH_KEY} ${DEPLOY_USER}@${DEPLOY_HOST} '
                        mkdir -p ${BACKUP_PATH}
                        TS=`date +%Y%m%d%H%M%S`
                        sudo tar -czf ${BACKUP_PATH}/html.bak.${TS}.tar.gz -C /var/www html || true
                    '

                    echo "Deploying new version..."
                    rsync -avz -e "ssh -i ${SSH_KEY}" --delete build/ ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}

                    echo "Restarting Apache..."
                    ssh -i ${SSH_KEY} ${DEPLOY_USER}@${DEPLOY_HOST} "sudo systemctl restart apache2"

                    echo "Deployment completed!"
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed! Attempting rollback..."
            sh '''
                PREV=$(ssh -i ${SSH_KEY} ${DEPLOY_USER}@${DEPLOY_HOST} "ls -1t ${BACKUP_PATH}/html.bak.*.tar.gz | head -n1 || true")
                if [ -n "$PREV" ]; then
                    echo "Restoring backup $PREV"
                    ssh -i ${SSH_KEY} ${DEPLOY_USER}@${DEPLOY_HOST} "sudo tar -xzf $PREV -C /; sudo systemctl restart apache2"
                else
                    echo "No backup available"
                fi
            '''
        }
        always {
            cleanWs()
        }
    }
}

