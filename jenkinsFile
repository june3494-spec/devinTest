pipeline {
  agent any

  environment {
    ARM_CLIENT_ID       = credentials('azure-client-id')
    ARM_CLIENT_SECRET   = credentials('azure-client-secret')
    ARM_TENANT_ID       = credentials('azure-tenant-id')
    ARM_SUBSCRIPTION_ID = credentials('azure-subscription-id')
  }

  options {
    timestamps()
    ansiColor('xterm')
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Terraform Init (Azure Backend)') {
      steps {
        sh '''
          terraform init -reconfigure \
            -backend-config="resource_group_name=rg-tfstate" \
            -backend-config="storage_account_name=sttfstate123" \
            -backend-config="container_name=tfstate" \
            -backend-config="key=devin-dev.tfstate"
        '''
      }
    }

    stage('Terraform Validate') {
      steps {
        sh 'terraform validate'
      }
    }

    stage('Terraform Plan') {
      steps {
        sh '''
          terraform plan \
            -var-file=env/dev.tfvars \
            -out=tfplan
        '''
      }
    }

    stage('Terraform Apply') {
      when {
        branch 'main'
      }
      steps {
        input message: 'Apply Terraform to Azure?', ok: 'Apply'
        sh 'terraform apply tfplan'
      }
    }
  }

  post {
    always {
      sh 'rm -f tfplan || true'
    }
    failure {
      echo 'Terraform pipeline failed'
    }
  }
}