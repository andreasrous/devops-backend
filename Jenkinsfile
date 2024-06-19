pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'git@github.com:andreasrous/devops-backend.git'
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

        stage('Install ansible prerequisites') {
            steps {
                sh '''
                    ansible-galaxy install geerlingguy.postgresql
                '''
            }
        }

        stage('Install postgres') {
            steps {
                sh '''
                    export ANSIBLE_CONFIG=~/workspace/ansible/ansible.cfg
                    ansible-playbook -i ~/workspace/ansible/hosts.yaml \
                        -l dbserver-vm ~/workspace/ansible/playbooks/postgres.yaml
                '''
            }
        }

        stage('Deploy spring boot app') {
            steps {
                sh '''
                    export ANSIBLE_CONFIG=~/workspace/ansible/ansible.cfg
                    ansible-playbook -i ~/workspace/ansible/hosts.yaml \
                        -l appserver-vm -e db_server_url=localhost \
                        -e deploy_nginx=false \
                        ~/workspace/ansible/playbooks/spring.yaml
                '''
            }
        }

        stage('Deploy frontend') {
            steps {
                sh '''
                    export ANSIBLE_CONFIG=~/workspace/ansible/ansible.cfg
                    ansible-playbook -i ~/workspace/ansible/hosts.yaml \
                        -l frontserver-vm -e branch=main \
                        -e backend_server_url=http://localhost:9090 \
                        -e '{"app": {"VITE_BACKEND": "http://192.168.56.104"}}' \
                        ~/workspace/ansible/playbooks/vuejs.yaml
                '''
            }
        }
    }
}
