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

        stage('Install Dependencies') {
            steps {
                sh 'pip3 install -r requirements.txt --break-system-packages'
           sh 'pip-audit --format html --output reports/pip_audit_report.html'
 
            }
        }

        stage('Run Tests') {
            steps {
                sh 'pytest'
            }
        }

        stage('SCA Scan') {
            steps {
                sh '''
                mkdir -p ${REPORTS_DIR}
                docker run --rm \
                  -v $PWD:/src \
                  owasp/dependency-check \
                  --project "TP-Jenkins" \
                  --scan /src \
                  --format HTML \
                  --out /src/${REPORTS_DIR}
                '''
                archiveArtifacts artifacts: '${REPORTS_DIR}/dependency-check-report.html', allowEmptyArchive: true
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
