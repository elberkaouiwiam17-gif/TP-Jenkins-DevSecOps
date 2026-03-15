pipeline {
    agent any

    environment {
        REPORTS_DIR = 'reports'
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
                python3 -m pip install --break-system-packages -r requirements.txt

                # Installer pip-audit pour l'utilisateur courant
                python3 -m pip install --break-system-packages --user pip-audit

                # Créer le dossier pour les rapports
                mkdir -p ${REPORTS_DIR}

                # Lancer le scan SCA avec pip-audit et générer le rapport JSON
                python3 -m pip_audit --format json --output ${REPORTS_DIR}/pip_audit_report.json
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

            // Archiver le rapport JSON pour consultation
            archiveArtifacts artifacts: '${REPORTS_DIR}/pip_audit_report.json', allowEmptyArchive: true

            // Optionnel : tu peux ajouter un convertisseur JSON → HTML ici si besoin
        }

        success {
            echo '✅ Build succeeded! Tous les tests et scans ont été exécutés.'
        }

        failure {
            echo '❌ Build failed! Vérifiez les tests ou les vulnérabilités détectées.'
        }
    }
}
