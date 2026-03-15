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
                pip3 install --break-system-packages -r requirements.txt
                pip3 install --break-system-packages pip-audit
                mkdir -p ${REPORTS_DIR}
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
        success {
            echo '✅ Build succeeded! Tous les tests et scans ont été exécutés.'
        }
        failure {
            echo '❌ Build failed! Vérifiez les tests ou les vulnérabilités détectées.'
        }
    }
}
