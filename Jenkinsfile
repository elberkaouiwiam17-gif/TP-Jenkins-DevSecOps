pipeline {
    agent any

    environment {
        // Dossier où seront stockés les rapports
        REPORTS_DIR = 'reports'
    }

    stages {

        // ----------------------------
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        // ----------------------------
        stage('Install Dependencies & SCA Scan') {
            steps {
                script {
                    // Installer les dépendances Python
                    sh 'python3 -m pip install --break-system-packages -r requirements.txt'

                    // Installer pip-audit
                    sh 'python3 -m pip install --break-system-packages --user pip-audit'

                    // Créer dossier des rapports
                    sh "mkdir -p ${REPORTS_DIR}"

                    // Lancer pip-audit
                    def pip_audit_status = sh(
                        script: "python3 -m pip_audit --format json --output ${REPORTS_DIR}/pip_audit_report.json",
                        returnStatus: true
                    )

                    echo "pip-audit exit code: ${pip_audit_status}"

                    // Afficher un résumé des vulnérabilités détectées
                    sh """
                    python3 - <<EOF
import json
try:
    with open("${REPORTS_DIR}/pip_audit_report.json") as f:
        data = json.load(f)
        vulns = data.get("vulnerabilities", [])
        if vulns:
            print("Vulnérabilités détectées:")
            for v in vulns:
                print(f"- {v.get('package')} : {v.get('id')} ({v.get('fix_version')})")
        else:
            print("Aucune vulnérabilité détectée")
except FileNotFoundError:
    print("Rapport pip-audit introuvable")
EOF
                    """

                    // Optionnel : échouer le build si des vulnérabilités sont détectées
                    if (pip_audit_status != 0) {
                        error "Vulnérabilités détectées par pip-audit"
                    }
                }
            }
        }

        // ----------------------------
        stage('Run Tests') {
            steps {
                script {
                    def test_status = sh(
                        script: "pytest --maxfail=5 --tb=short",
                        returnStatus: true
                    )

                    echo "Pytest exit code: ${test_status}"

                    if (test_status != 0) {
                        error "Les tests ont échoué"
                    }
                }
            }
        }

        // ----------------------------
        stage('SAST Scan') {
            steps {
                script {
                    // Avec SonarQube : utilise le scanner configuré dans Jenkins
                    // Assurez-vous que :
                    // 1) Le serveur SonarQube est accessible depuis l'agent (ex: http://host.docker.internal:9000)
                    // 2) Le token d'authentification est configuré dans Jenkins
                    withSonarQubeEnv('sonar-scanner') {
                        sh "${tool 'sonar-scanner'}/bin/sonar-scanner -Dsonar.projectKey=TP-Jenkins -Dsonar.sources=."
                    }
                }
            }
        }

    } // fin stages

    post {
        always {
            echo 'Archivage des rapports...'
            archiveArtifacts artifacts: 'reports/pip_audit_report.json', allowEmptyArchive: true
            archiveArtifacts artifacts: '**/reports/*.xml', allowEmptyArchive: true
        }

        success {
            echo 'Build terminé avec succès'
        }

        failure {
            echo 'Build terminé avec des problèmes détectés (tests ou vulnérabilités)'
        }
    }
}
