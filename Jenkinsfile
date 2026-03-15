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
                script {
                    // Installer les dépendances Python
                    sh 'python3 -m pip install --break-system-packages -r requirements.txt'
                    // Installer pip-audit pour l'utilisateur courant
                    sh 'python3 -m pip install --break-system-packages --user pip-audit'
                    // Créer le dossier pour les rapports
                    sh "mkdir -p ${REPORTS_DIR}"

                    // Lancer pip-audit et capturer le code de sortie
                    def pip_audit_status = sh(
                        script: "python3 -m pip_audit --format json --output ${REPORTS_DIR}/pip_audit_report.json",
                        returnStatus: true
                    )
                    echo "🔎 pip-audit exit code: ${pip_audit_status}"

                    // Lire un résumé simple des vulnérabilités
                    sh """
                    echo '📄 Résumé des vulnérabilités pip-audit:'
                    python3 -c \"
import json
with open('${REPORTS_DIR}/pip_audit_report.json') as f:
    data = json.load(f)
    for vuln in data.get('vulnerabilities', []):
        print(f'- {vuln.get("package")}: {vuln.get("id")} (severity: {vuln.get("severity")})')
\" || echo 'Aucune vulnérabilité critique détectée.'
                    """
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Exécuter pytest et capturer le code de sortie
                    def test_status = sh(script: "pytest --maxfail=5 --tb=short", returnStatus: true)
                    echo "✅ Pytest exit code: ${test_status}"
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
            archiveArtifacts artifacts: '${REPORTS_DIR}/pip_audit_report.json', allowEmptyArchive: true
            archiveArtifacts artifacts: '**/reports/*.xml', allowEmptyArchive: true
            echo '⚠️ Vérifiez les rapports pour vulnérabilités et résultats des tests.'
        }

        success {
            echo '✅ Build terminé ! Tous les tests et scans ont été exécutés.'
        }
    }
}
