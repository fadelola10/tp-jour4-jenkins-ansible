pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        ANSIBLE_FORCE_COLOR        = 'true'
    }

    parameters {
        choice(
            name: 'ENV',
            choices: ['dev', 'prod'],
            description: 'Environnement cible'
        )
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
                    sh "ansible all -i inventories/${params.ENV}/hosts.ini -m ping"
                }
            }
        }

        stage('Approval (prod)') {
            when { expression { params.ENV == 'prod' } }
            steps {
                input message: 'Déployer en PROD ?', ok: 'GO'
            }
        }

        stage('Deploy') {
            steps {
                dir('ansible') {
                    sh "ansible-playbook -i inventories/${params.ENV}/hosts.ini site.yml"
                }
            }
        }

        stage('Smoke Test') {
            steps {
                dir('ansible') {
                    sh """
                        TARGET=\$(grep -A1 '\\[web\\]' inventories/${params.ENV}/hosts.ini | tail -1 | awk '{print \$1}')
                        echo "Test sur \$TARGET..."
                        docker exec \$TARGET curl -f -s -o /dev/null -w "HTTP %{http_code}\\n" http://localhost
                    """
                }
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
