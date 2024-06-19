pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '30'))
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'git@github.com:andreasrous/devops-backend.git'
            }
        }

        stage('Test') {
            steps {
                sh './mvnw test'
            }
        }

        stage('Run ansible pipeline') {
            steps {
                build job: 'ansible'
            }
        }

        stage('Install project with docker compose') {
            steps {
                sh '''
                    export ANSIBLE_CONFIG=~/workspace/ansible/ansible.cfg
                    ansible-playbook -i ~/workspace/ansible/hosts.yaml -l deploy-vm ~/workspace/ansible/playbooks/spring-vue-docker.yaml
                '''
            }
        }
    }
}