pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        SRVWEB = '100.95.221.197'
        SRVDATA = '100.95.95.30'
        WEB_SERVER = "$SRVWEB"  // Variable pour les vérifications curl
        INVENTORY = 'ansible/hosts'
        PLAYBOOK = 'ansible/playbook.yml'
        ROLLBACK = 'ansible/rollback.yml'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Branche : ${env.GIT_BRANCH}"
                echo "Commit  : ${env.GIT_COMMIT}"
            }
        }

        stage('Vérification Ansible') {
            steps {
                sh 'ansible --version'
                sh 'ansible-playbook --syntax-check -i $INVENTORY $PLAYBOOK'
            }
        }

        stage('Test connectivité VMs') {
            steps {
                sh 'ansible all -i $INVENTORY -m ping'
            }
        }

        stage('Déploiement Ansible') {
            steps {
                sh '''
                ansible-playbook \
                  -i $INVENTORY \
                  $PLAYBOOK \
                  -v
                '''
            }
        }

        stage('Vérification déploiement') {
            steps {
                sh '''
                # Vérifier que la page d'accueil est accessible et contient le texte attendu
                curl -f http://$WEB_SERVER | grep -q "Bienvenue sur l\'application métier" || exit 1
                # Vérifier que la calculatrice est accessible et contient le titre attendu
                curl -f http://$WEB_SERVER/calculatrice.html | grep -q "<h2>Calculatrice</h2>" || exit 1
                '''
                echo 'Serveur web et calculatrice déployés correctement'
            }
        }
    }

    post {
        success {
            echo "Déploiement réussi sur ${env.GIT_BRANCH}"
        }

        failure {
            echo 'Échec — lancement du rollback Ansible'
            sh '''
            ansible-playbook \
              -i $INVENTORY \
              $ROLLBACK \
              -v
            '''
        }

        always {
            echo "Pipeline terminé — statut : ${currentBuild.currentResult}"
        }
    }
}
