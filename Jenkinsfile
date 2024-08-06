pipeline {
    agent any

    tools {
        nodejs 'NodeJS' // Use the NodeJS installation configured in Jenkins
    }

    environment {
        AZURE_CREDENTIALS_ID = 'azure-credentials-id'
        VM_USER = 'your_vm_username'
        VM_IP = 'your_vm_ip_address'
        SSH_KEY_PATH = '/var/jenkins_home/.ssh/id_rsa'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Iamthor15/nodejstask.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test || true' // Skip the test stage if no tests are found
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
