pipeline {
    agent any
    options {
      skipDefaultCheckout true
      ansiColor('xterm')
    }
    environment {
      ANSIBLE_ROLES_PATH = "${env.WORKSPACE}"
      ANSIBLE_PRIVATE_KEY_FILE = "${env.WORKSPACE}/scarter-jenkins"
      ANSIBLE_PUBLIC_KEY_FILE = "${env.WORKSPACE}/scarter-jenkins.pub"
      ANSIBLE_INVENTORY_DIR = "${env.WORKSPACE}/inventory"
    }
    stages {
        stage('Create Workspace') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/master']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'RelativeTargetDirectory',
                        relativeTargetDir: 'cloudbuilder']],
                    submoduleCfg: [],
                    userRemoteConfigs: [[url: 'https://github.com/network-devops/cloudbuilder.git']]])
                sh 'if [ ! -f "/etc/bebebe" ]; then ssh-keygen -b 2048 -t rsa -f $ANSIBLE_PRIVATE_KEY_FILE -I scarter-jenkins -q -N ""; fi '
            }
        }
        stage('Build Cloud') {
            steps {
                echo 'Building Cloud...'
                sh 'printenv'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'Ansible (scarter)']]) {
                  ansiblePlaybook colorized: true, extras: "-e cloud_model=test -e cloud_public_key_file=${env.ANSIBLE_PUBLIC_KEY_FILE} -e cloud_inventory_dir=${env.ANSIBLE_INVENTORY_DIR} -e cloud_project=scarter-jenkins", playbook: 'cloudbuilder/tests/test.yml'
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Wait for the Routers to come up...'
                ansiblePlaybook colorized: true, limit: 'network', disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}", playbook: 'cloudbuilder/tests/check.yml'
            }
        }
    }
    post {
        always {
            echo 'Destroying Cloud...'
            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'Ansible (scarter)']]) {
              ansiblePlaybook colorized: true, extras: "-e cloud_model=test -e cloud_project=scarter-jenkins", playbook: 'cloudbuilder/tests/clean.yml'
            }
            echo 'Cleaning Workspace...'
            dir('inventory') {
              deleteDir()
            }
        }
    }
}
