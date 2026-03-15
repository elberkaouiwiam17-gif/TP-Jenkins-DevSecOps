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

                # Lancer le scan SCA et générer le rapport JSON
                pip-audit --format json --output ${REPORTS_DIR}/pip_audit_report.json
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

            // Archiver le rapport JSON pour téléchargement
            archiveArtifacts artifacts: '${REPORTS_DIR}/pip_audit_report.json', allowEmptyArchive: true

            // Optionnel : publier comme HTML en convertissant JSON → tableau lisible via script
            // publishHTML(...) si tu ajoutes un convertisseur
        }

        success {
            echo '✅ Build succeeded! Tous les tests et scans ont été exécutés.'
        }

        failure {
            echo '❌ Build failed! Vérifiez les tests ou les vulnérabilités détectées.'
        }
    }
}
