pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
               checkout scm  }
        }
        stage('Install Dependencies') {
            steps {
                sh 'pip3 install -r requirements.txt --break-system-packages'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'pytest'
            }
        }
        stage('SCA Scan') {
            steps {
                sh 'dependency-check.sh --project "TP-Jenkins" --scan . --format HTML --out reports'
                archiveArtifacts artifacts: 'reports/dependency-check-report.html', allowEmptyArchive: true
            }
        }
        stage('SAST Scan') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh 'sonar-scanner -Dsonar.projectKey=TP-Jenkins -Dsonar.sources=.'
                }
            }
        }
    } // <-- fin de stages
    post {   // <-- post doit être ici, PAS à l'intérieur d'un stage
        failure {
            echo 'Build failed due to errors or vulnerabilities'
        }
    }
}
