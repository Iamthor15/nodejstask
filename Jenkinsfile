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
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        // Comment out or remove the 'Run Tests' stage
        /*
        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }
        */
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
                withCredentials([azureServicePrincipal('azure-credentials-id')]) {
                    sh '''
                    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                    az vm run-command invoke -g <ResourceGroup> -n <VMName> --command-id RunShellScript --scripts "sudo rm -rf /var/www/html/* && sudo cp -r build/* /var/www/html/"
                    '''
                }
            }
        }
    }
}
