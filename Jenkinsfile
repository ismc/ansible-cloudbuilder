pipeline {
    agent any
    options {
      disableConcurrentBuilds()
      ansiColor('xterm')
    }
    environment {
      ANSIBLE_ROLES_PATH = "${env.WORKSPACE}"
      ANSIBLE_INVENTORY_DIR = "${env.WORKSPACE}/inventory"
    }
    stages {
        stage('Prepare Workspace') {
            steps {
                sh 'ln -sf $PWD cloudbuilder'
            }
        }
        stage('Build Cloud') {
            steps {
                echo 'Building Cloud...'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'Ansible (scarter)']]) {
                  ansiblePlaybook colorized: true, extras: "-e cloud_model=test -e cloud_inventory_dir=${env.ANSIBLE_INVENTORY_DIR} -e cloud_instance=jenkins-cloudbuilder -e cloud_project=scarter-jenkins -e cloud_key_name=scarter-jenkins", playbook: 'cloudbuilder/tests/build.yml'
                }
            }
        }
        stage('Run Tests') {
            steps {
                echo 'Running Validation Tests...'
                  ansiblePlaybook credentialsId: 'scarter-jenkins_key', colorized: true, limit: 'network', disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}", playbook: 'cloudbuilder/tests/test.yml'

            }
        }
        stage('Destroy Cloud') {
            steps {
                echo 'Destroying Cloud...'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'Ansible (scarter)']]) {
                    ansiblePlaybook colorized: true, extras: "-e cloud_model=test -e cloud_instance=jenkins-cloudbuilder -e cloud_project=scarter-jenkins", playbook: 'cloudbuilder/tests/destroy.yml'
                }
            }
        }
    }
    post {
        always {
            echo 'Cleaning Workspace...'
            dir('inventory') {
              deleteDir()
            }
        }
    }
}
