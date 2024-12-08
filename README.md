pipeline {
    agent any
    stages {
        stage('Cloner le dépôt GitHub') {
            steps {
                git 'https://github.com/<votre-utilisateur>/ynov-html-deployment.git'
            }
        }
        stage('Déployer sur test.ynov.local') {
            steps {
                sh 'cp index.html /var/www/test.ynov.local/'
            }
        }
        stage('Tester la page HTML') {
            steps {
                script {
                    def response = sh(script: "curl -o /dev/null -s -w '%{http_code}' http://test.ynov.local", returnStdout: true).trim()
                    if (response != '200') {
                        error "Erreur HTTP détectée : ${response}"
                    }
                }
            }
        }
        stage('Déployer sur ynov.local') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                sh 'cp index.html /var/www/ynov.local/'
            }
        }
    }
}
