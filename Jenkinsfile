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
                # || true empêche le build de bloquer si des vulnérabilités sont trouvées
                python3 -m pip_audit --format json --output ${REPORTS_DIR}/pip_audit_report.json || true
                '''
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Exécuter pytest et capturer le code de sortie
                    def test_status = sh(script: "pytest", returnStatus: true)
                    echo "Pytest exit code: ${test_status}"

                    // Tu peux aussi créer un fichier de rapport HTML si tu veux
                }
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
            echo '📄 Archivage des rapports...'

            // Archiver le rapport pip-audit JSON
            archiveArtifacts artifacts: '${REPORTS_DIR}/pip_audit_report.json', allowEmptyArchive: true

            // Archiver tous les fichiers pytest si tu en génères
            archiveArtifacts artifacts: '**/reports/*.xml', allowEmptyArchive: true
        }

        success {
            echo '✅ Build terminé ! Tous les tests et scans ont été exécutés.'
        }

        failure {
            echo '❌ Build terminé avec des problèmes détectés (tests ou vulnérabilités).'
        }
    }
}
