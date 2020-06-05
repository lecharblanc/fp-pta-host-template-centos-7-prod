pipeline {
    agent {
        label 'pta-controller'
    }
    options {
        disableConcurrentBuilds()
    }
    parameters {
        booleanParam(name: "Terraform_Init_Local", defaultValue: false, description: 'Perform a terraform init with the already downloaded plugins in /usr/lib/custom-terraform-plugins (i.e. Local) or download them anew from the internet (i.e. External).')
    }
    //environment {
    // Uncomment TF_LOG if you're trying to debug a problem with Terraform.
    //TF_LOG = "trace"
    //}
    stages {
        stage('Terraform Init Local') {
            when {
                expression { params.Terraform_Init_Local }
            }
            steps {
                withCredentials([
                        usernamePassword(credentialsId: 'terraform-vsphere', usernameVariable: 'TF_VAR_vsphere_user', passwordVariable: 'TF_VAR_vsphere_password'),
                        usernamePassword(credentialsId: 'slcartifactory', usernameVariable: 'ARTIFACTORY_USERNAME', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    sh 'terraform init -no-color -upgrade -plugin-dir=/usr/lib/custom-terraform-plugins'
                }
            }
        }
        stage('Terraform Init External') {
            when {
                expression { ! params.Terraform_Init_Local }
            }
            steps {
                withCredentials([
                        usernamePassword(credentialsId: 'terraform-vsphere', usernameVariable: 'TF_VAR_vsphere_user', passwordVariable: 'TF_VAR_vsphere_password'),
                        usernamePassword(credentialsId: 'slcartifactory', usernameVariable: 'ARTIFACTORY_USERNAME', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    sh 'terraform init -no-color -upgrade'
                }
            }
        }
        stage('Terraform Plan') {
            steps {
                withCredentials([
                        usernamePassword(credentialsId: 'terraform-vsphere', usernameVariable: 'TF_VAR_vsphere_user', passwordVariable: 'TF_VAR_vsphere_password'),
                        usernamePassword(credentialsId: 'slcartifactory', usernameVariable: 'ARTIFACTORY_USERNAME', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    sh 'terraform plan -no-color'
                }
            }
        }
        stage('Continue?') {
            steps {
                input 'Are you sure you want to continue with the apply?'
            }
        }
        stage('Terraform Apply') {
            steps {
                withCredentials([
                        usernamePassword(credentialsId: 'terraform-vsphere', usernameVariable: 'TF_VAR_vsphere_user', passwordVariable: 'TF_VAR_vsphere_password'),
                        usernamePassword(credentialsId: 'slcartifactory', usernameVariable: 'ARTIFACTORY_USERNAME', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    sh 'terraform apply -no-color -auto-approve'
                }
            }
        }
        stage('Shutdown') {
            steps {
                withCredentials([
                        usernamePassword(credentialsId: 'terraform-vsphere', usernameVariable: 'TF_VAR_vsphere_user', passwordVariable: 'TF_VAR_vsphere_password'),
                        usernamePassword(credentialsId: 'slcartifactory', usernameVariable: 'ARTIFACTORY_USERNAME', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    sh '''
                        virtualenv virt_ansible
                        source virt_ansible/bin/activate
                        pip install ansible
                        ansible-playbook -i inventory.py shutdown.yml
                        deactivate
                       '''
                }
            }
        }
        // TODO: At some point it might be nice to parse the ip address or other things from the terraform output and include it in the email.
        stage('Terraform Output') {
            steps {
                withCredentials([
                        usernamePassword(credentialsId: 'terraform-vsphere', usernameVariable: 'TF_VAR_vsphere_user', passwordVariable: 'TF_VAR_vsphere_password'),
                        usernamePassword(credentialsId: 'slcartifactory', usernameVariable: 'ARTIFACTORY_USERNAME', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                    sh 'terraform output'
                }
            }
        }
    }
    post {
        failure {
            emailext body: '''$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS<br><br>Check the console output at ${BUILD_URL}console to view the results.''', mimeType: 'text/html', recipientProviders: [requestor()], subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: "pta.admin@company.com"
        }
    }
}