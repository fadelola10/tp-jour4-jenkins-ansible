pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        ANSIBLE_FORCE_COLOR        = 'true'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                sh 'ls -la'
            }
        }

        stage('Lint Ansible') {
            steps {
                dir('ansible') {
                    sh 'ansible-playbook site.yml --syntax-check'
                }
            }
        }

        stage('Dry Run') {
            steps {
                dir('ansible') {
                    sh 'ansible all -m ping'
                }
            }
        }

        stage('Deploy') {
            steps {
                dir('ansible') {
                    sh 'ansible-playbook -i inventory.ini site.yml'
                }
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    echo "Test du site sur target1..."
                    docker exec target1 curl -f -s -o /dev/null -w "HTTP %{http_code}\n" http://localhost
                '''
            }
        }
    }

    post {
        always {
            echo "Job ${currentBuild.fullDisplayName} terminé avec statut: ${currentBuild.currentResult}"
        }
        success {
            echo '✅ Déploiement réussi'
        }
        failure {
            echo '❌ Déploiement échoué — vérifier les logs'
        }
    }
}
