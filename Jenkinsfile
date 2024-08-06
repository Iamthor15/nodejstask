pipeline {
    agent any

    tools {
        nodejs 'NodeJS' // Use the NodeJS installation configured in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Iamthor15/nodejstask.git'
            }
        }

        stage('Verify File Structure') {
            steps {
                sh '''
                echo "Listing files in the workspace..."
                ls -la
                echo "Listing files in the src directory..."
                ls -la src
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'build/**', fingerprint: true
            }
        }

        stage('Deploy to Azure VM') {
            steps {
                script {
                    // Set up SSH keys
                    sh '''
                    if [ ! -f $SSH_KEY_PATH ]; then
                        ssh-keygen -t rsa -b 2048 -f $SSH_KEY_PATH -N ""
                    fi
                    ssh-copy-id -i $SSH_KEY_PATH.pub $VM_USER@$VM_IP
                    '''
                    
                    // Use Azure CLI to deploy application
                    withCredentials([azureServicePrincipal(credentialsId: "${AZURE_CREDENTIALS_ID}", clientIdVariable: 'AZURE_CLIENT_ID', clientSecretVariable: 'AZURE_CLIENT_SECRET', tenantIdVariable: 'AZURE_TENANT_ID', subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID')]) {
                        sh '''
                        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                        az vm run-command invoke -g <ResourceGroup> -n <VMName> --command-id RunShellScript --scripts "
                            sudo apt-get update
                            sudo apt-get install -y nginx
                            sudo apt-get install -y nodejs npm
                            sudo rm -rf /var/www/html/*
                            sudo mkdir -p /var/www/html
                            sudo mkdir -p /var/www/html/build
                            scp -i $SSH_KEY_PATH -r build/* $VM_USER@$VM_IP:/var/www/html/
                            sudo systemctl restart nginx
                        "
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean workspace after build
        }
    }
}
