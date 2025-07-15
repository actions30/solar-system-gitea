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
        stage('NPM Dependency Audit'){
          steps{
            sh '''
               npm audit --audit-level=critical
               echo $?
               '''
           }
        }    
    }
} 

