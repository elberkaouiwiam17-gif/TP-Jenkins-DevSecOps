pipeline {
    agent any

    stages {

        stage('Checkout Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/elberkaouiwiam17-gif/TP-Jenkins-DevSecOps.git',
                    gitTool: 'GitLatest'
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

    }
}
