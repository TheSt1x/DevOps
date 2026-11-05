pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'false'
        ANSIBLE_FORCE_COLOR = 'true'
        HOST_USER = 'vboxuser'
        PROJECT_DIR = '/home/vboxuser/Desktop/DevOps/Ansible'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Проверка содержимого рабочей директории...'
                sh 'ls -la'
            }
        }

        stage('Prepare Environment') {
            steps {
                echo 'Этап 1: Установка ПО и клонирование репозиториев...'
                sshagent(credentials: ['ansible-node-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${HOST_USER}@host.docker.internal "echo ok"
                        cd ${PROJECT_DIR} && \
                        source ~/ansible-venv/bin/activate && \
                        ansible-playbook -i hosts-jenkins jenkins-playbook.yml --tags prepare
                    '''
                }
            }
        }

        stage('Start Applications') {
            steps {
                echo 'Этап 2: Запуск контейнеров...'
                sshagent(credentials: ['ansible-node-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${HOST_USER}@host.docker.internal "echo ok"
                        cd ${PROJECT_DIR} && \
                        source ~/ansible-venv/bin/activate && \
                        ansible-playbook -i hosts-jenkins jenkins-playbook.yml --tags deploy
                    '''
                }
            }
        }
    }

    post {
        success { echo 'Пайплайн успешно завершен!' }
        failure { echo 'Ошибка выполнения пайплайна.' }
    }
}
