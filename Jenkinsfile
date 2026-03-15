pipeline {
    agent any

    environment {
        REPORTS_DIR = 'reports'
        // Ajouter ~/.local/bin au PATH pour pip-audit
        PATH = "${env.HOME}/.local/bin:${env.PATH}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies & SCA Scan') {
            steps {
                sh '''
                # Installer les dépendances Python
                pip3 install --break-system-packages -r requirements.txt

                # Installer pip-audit
                pip3 install --break-system-packages pip-audit

                # Créer le dossier pour les rapports
                mkdir -p ${REPORTS_DIR}

                # Lancer le scan SCA et générer le rapport HTML
                pip-audit --format html --output ${REPORTS_DIR}/pip_audit_report.html
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh 'pytest'
            }
        }

        stage('SAST Scan') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh 'sonar-scanner -Dsonar.projectKey=TP-Jenkins -Dsonar.sources=.'
                }
            }
        }
    }

    post {
        always {
            echo '📄 Archivage du rapport SCA pip-audit...'

            archiveArtifacts artifacts: '${REPORTS_DIR}/pip_audit_report.html', allowEmptyArchive: true

            publishHTML(target: [
                reportDir: "${REPORTS_DIR}",
                reportFiles: 'pip_audit_report.html',
                reportName: 'Rapport SCA Python',
                alwaysLinkToLastBuild: true,
                keepAll: true
            ])
        }

        success {
            echo '✅ Build succeeded! Tous les tests et scans ont été exécutés.'
        }

        failure {
            echo '❌ Build failed! Vérifiez les tests ou les vulnérabilités détectées.'
        }
    }
}
