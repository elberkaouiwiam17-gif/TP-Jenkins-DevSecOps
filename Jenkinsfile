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

                    // Afficher résumé des vulnérabilités
                    sh """
                    echo 'Résumé des vulnérabilités pip-audit:'
                    python3 - <<EOF
import json
try:
    with open("${REPORTS_DIR}/pip_audit_report.json") as f:
        data = json.load(f)
        vulns = data.get("vulnerabilities", [])
        if vulns:
            for v in vulns:
                print(f"- {v.get('package')} : {v.get('id')}")
        else:
            print("Aucune vulnérabilité détectée")
except:
    print("Rapport pip-audit introuvable")
EOF
                    """
                }
            }
        }

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

      stage('SAST Scan') {
    steps {
        script {
            def scannerHome = tool 'sonar-scanner'
            withSonarQubeEnv('MySonarQubeServer') {
                sh """
                ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=TP-Jenkins \
                -Dsonar.sources=. \
                -Dsonar.host.url=$SONAR_HOST_URL \
                -Dsonar.login=$SONAR_AUTH_TOKEN
                """
            }
        }
    }
}

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
