pipeline {
    agent any

    stages {

        stage('Clone Repository') {
            steps {
               git branch: 'main', url: 'https://github.com/elberkaouiwiam17-gif/TP-Jenkins-DevSecOps.git'  }
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
