pipeline {
    agent any

    tools {
        nodejs 'Node 24.4.0'
    }

    stages {
        stage('VM Node version') {
            steps {
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}

