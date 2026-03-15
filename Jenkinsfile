pipeline {
    agent any

    stages {

       stage('Checkout Repository') {
    steps {
        checkout scm
    }
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
                sh 'dependency-check.sh --project "TP-Jenkins-DevSecOps" --scan . --format HTML'
            }
        }
        stage('SAST Scan') {
            steps {
                sh 'sonar-scanner'
            }
        }
    }
    post {
        failure {
            echo 'Build failed due to errors or vulnerabilities'
        }
    }
    }
}
