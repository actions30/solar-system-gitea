pipeline {
    agent any

    tools {
        nodejs 'Node 24.4.0'
    }

    stages {
        stage('Installing Dependencies') {
            steps {
                sh ' npm install --no-audit'
            }
        }
    }
}

