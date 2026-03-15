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

        stage('SAST Scan') {
    steps {
        withSonarQubeEnv('MySonarQubeServer') { // Nom du serveur configuré
            sh 'sonar-scanner -Dsonar.projectKey=TP-Jenkins -Dsonar.sources=.'
        }
    }
}
    }
    
    }

