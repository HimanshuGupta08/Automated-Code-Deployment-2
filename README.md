**Jenkins Deployment Pipeline for Code-Store Frontend**

This repository contains a Jenkins pipeline that automates the deployment process for Next.js Frontend project. It supports deployment to both a local Jenkins server for development (dev branch) and a production server for the production (production branch).

**Overview**

**The pipeline performs the following steps:**

**Clone the repository:** It clones the dev and production branches from the GitHub repository to the Jenkins workspace.

**Install dependencies and build:** The pipeline installs required dependencies, builds the project, and deploys it based on the branch.

**Deploy to servers:**

**Development (dev branch)**: Deploys to the local Jenkins server.

**Production (production branch)**: Deploys to the production server via SSH.

**Process management**: Uses PM2 to manage the application process, restarting it if it exists, or starting a new process if not.

**Pipeline Flow**

**The Jenkins pipeline consists of two main stages:**

**Clone Repository:** Clones the repository from GitHub.

**Check Branch, Install, Build and Deploy:** This stage checks the branch name (dev or production), installs dependencies, builds the project, and deploys it accordingly.

**Prerequisites**

**Jenkins:** A running Jenkins instance.

**GitHub:** A GitHub repository containing the frontend project.

**PM2:** A process manager for Node.js applications.

**SSH Credentials**: SSH access to the production server.

**Configuration**

**1. Environment Variables**

The pipeline uses the following environment variables:

DEPLOY_DIR: Directory where the code will be deployed on the server.

GITHUB_REPO: The GitHub repository URL to clone.

Update these variables in the pipeline according to your deployment setup.

**2. Jenkins Credentials**

**github-credentials:** Jenkins credentials for GitHub access.

**prod-server-ssh:** SSH credentials for accessing the production server.

Ensure these credentials are configured in Jenkins under Manage Jenkins > Manage Credentials.

**3. Branches**

The pipeline currently supports two branches:

**dev:** For local development deployment.

**production:** For deploying to the production server.

**4. PM2 Process Management**

The pipeline uses PM2 to manage the frontend application:

If the process is already running, it restarts the existing process.

If the process is not running, it installs PM2 globally and starts a new process.

**Pipeline Steps**

**1. Clone Repository**

checkout([

    $class: 'GitSCM',
    
    branches: [[name: '*/dev'], [name: '*/production']],
    
    userRemoteConfigs: [[
    
        url: GITHUB_REPO,
        
        credentialsId: 'github-credentials'
        
    ]]
    
])

**2. Branch Check, Install, Build and Deploy**

**For dev:**

cd ${DEPLOY_DIR} || exit 1

git pull origin dev || exit 1

npm install || exit 1

npm run build || exit 1

# Check and manage PM2 process

pm2 describe codestorefrontend || pm2 start npm --name codestorefrontend -- run start

**For production:**

ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} user@production-server << 'EOF'

    cd ${DEPLOY_DIR} || exit 1
    
    git pull origin production || exit 1
    
    npm install || exit 1
    
    npm run build || exit 1
    
    # Check and manage PM2 process
    
    pm2 describe codestorefrontend || pm2 start npm --name codestorefrontend -- run start
    
EOF

**3. Post-build Actions**
   
**After the build:**

**Success:** The pipeline will print a success message.

**Failure:** If the build fails, Jenkins will print an error message.

**Always: **Cleans up the workspace after the build.

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

**Troubleshooting**

**PM2 Issues:** Ensure that PM2 is correctly installed on the server.

**SSH Connection Issues:** Verify that the SSH credentials are correctly configured in Jenkins.

**Build Errors:** Check the logs for specific errors related to npm install or npm run build.
