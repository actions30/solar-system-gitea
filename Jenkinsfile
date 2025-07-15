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

        stage('Dependency scanning'){
          paralell {
                stage('NPM Dependency Audit'){
                  steps{
                    sh '''
                      npm audit --audit-level=critical
                      echo $?
                      '''
                  }
                }
                stage('OWASP Dependency Check'){
                  steps{
                    dependencyCheck additionalArguments: '''
                    --scan \'./\'
                    --out \'./\'
                    --format \'ALL\'
                    --prettyPrint ''', odcInstallation: 'OWSAP-DepCheck-10'
                    dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true

          
                }

                }
          }
        }    
    }
} 

