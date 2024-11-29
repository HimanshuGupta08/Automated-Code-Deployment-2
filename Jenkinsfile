pipeline {
    agent any
    environment {
        DEPLOY_DIR = '/var/www/html/code-storefrontend' // Deployment directory
        GITHUB_REPO = 'https://github.com/richest/code-store-frontend.git'
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository to Jenkins workspace...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/dev'], [name: '*/production']],
                    userRemoteConfigs: [[
                        url: GITHUB_REPO,
                        credentialsId: 'github-credentials'
                    ]]
                ])
            }
        }

        stage('Check Branch, Install, Build and Deploy') {
            steps {
                script {
                    def branch = env.BRANCH_NAME
                    echo "Triggered build for branch: ${branch}"

                    if (branch == 'dev') {
                        echo "Deploying to local Jenkins server..."
                        // Local deployment steps (same server as Jenkins)
                        sh """
                            cd ${DEPLOY_DIR} || exit 1
                            git pull origin dev || exit 1
                            npm install || exit 1
                            npm run build || exit 1
                            
                            # Check if the PM2 process exists
                            if pm2 describe codestorefrontend > /dev/null 2>&1; then
                                echo "Process exists, restarting..."
                                pm2 restart codestorefrontend || exit 1
                            else
                                echo "Process does not exist, installing PM2 and starting new process..."
                                npm install pm2@latest -g || exit 1
                                pm2 start npm --name codestorefrontend -- run start || exit 1
                            fi
                        """
                    } else if (branch == 'production') {
                        echo "Deploying to production server via SSH..."
                        withCredentials([sshUserPrivateKey(credentialsId: 'prod-server-ssh', keyFileVariable: 'SSH_KEY')]) {
                            sh """
                                ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} user@production-server << 'EOF'
                                    cd ${DEPLOY_DIR} || exit 1
                                    git pull origin production || exit 1
                                    npm install || exit 1
                                    npm run build || exit 1
                                    
                                    # Check if the PM2 process exists
                                    if pm2 describe codestorefrontend > /dev/null 2>&1; then
                                        echo "Process exists, restarting..."
                                        pm2 restart codestorefrontend || exit 1
                                    else
                                        echo "Process does not exist, installing PM2 and starting new process..."
                                        npm install pm2@latest -g || exit 1
                                        pm2 start npm --name codestorefrontend -- run start || exit 1
                                    fi
                                EOF
                            """
                        }
                    } else {
                        error "Unknown branch: ${branch}. Deployment aborted."
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed. Check the logs for more details.'
        }
        always {
            cleanWs() // Cleans workspace after build
        }
    }
}
