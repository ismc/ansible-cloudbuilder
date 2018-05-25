pipeline {
    agent any
    options {
      skipDefaultCheckout true
    }
    stages {
        stage('Create Workspace') {
            steps {
                dir ('roles') {
                  git url: 'https://github.com/network-devops/cloudbuilder.git'
                }
            }
        }
        stage('Build Cloud') {
            steps {
                echo 'Building Cloud...'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'Ansible (scarter)']]) {
                  ansiblePlaybook colorized: true, disableHostKeyChecking: true, extras: '-e cloud_model=an-demo1 -e cloud_project=scarter-jenkins', playbook: 'build-demo.yml'
                }
            }
        }
        stage('Run Tests') {
            steps {
                echo 'Wait for the Routers to come up...'
                ansiblePlaybook colorized: true, limit: 'network', disableHostKeyChecking: true, inventory: 'inventory/scarter-jenkins', playbook: 'common/check-ssh.yml'

                echo 'Running network-system.yml...'
                ansiblePlaybook colorized: true, disableHostKeyChecking: true, inventory: 'inventory/scarter-jenkins', playbook: 'net/network-system.yml'

                echo 'Running network-security.yml...'
                ansiblePlaybook colorized: true, disableHostKeyChecking: true, inventory: 'inventory/scarter-jenkins', playbook: 'net/network-security.yml'

                echo 'Running network-interfaces.yml...'
                ansiblePlaybook colorized: true, disableHostKeyChecking: true, inventory: 'inventory/scarter-jenkins', playbook: 'net/network-interfaces.yml'

                echo 'Running network-ipsec-vpn.yml...'
                ansiblePlaybook colorized: true, disableHostKeyChecking: true, inventory: 'inventory/scarter-jenkins', playbook: 'net/network-ipsec-vpn.yml'

                echo 'Running network-bgp.yml...'
                ansiblePlaybook colorized: true, disableHostKeyChecking: true, inventory: 'inventory/scarter-jenkins', playbook: 'net/network-bgp.yml'

                echo 'Running network-check.yml...'
                ansiblePlaybook colorized: true, disableHostKeyChecking: true, inventory: 'inventory/scarter-jenkins', playbook: 'net/network-check.yml'
            }
        }
    }
/*
    post {
        always {
            echo 'Destroying Cloud...'
            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'Ansible (scarter)']]) {
              ansiblePlaybook colorized: true, disableHostKeyChecking: true, extras: '-e cloud_model=an-demo1 -e cloud_project=scarter-jenkins', playbook: 'destroy-demo.yml'
            }
            echo 'Cleaning Workspace...'
            deleteDir()
        }
    }
*/
}
