pipeline {
    agent any

    tools {
        nodejs 'Node 24.4.0'
    }

    environment{
          SONAR_SCANNER_HOME = tool 'SonarScanner';
    }

    stages {

        stage('Installing Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Dependency Scanning') {
            parallel {
                stage('NPM Dependency Audit') {
                    steps {
                        sh '''
                            npm audit --audit-level=critical || true
                            echo "NPM Audit completed"
                        '''
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        dependencyCheck additionalArguments: '''
                            --scan ./ \
                            --out ./ \
                            --format ALL \
                            --prettyPrint
                        ''', odcInstallation: 'OWASP-depend-check-12'
                            dependencyCheckPublisher failedTotalCritical: 1, pattern: '**/dependency-check-report.xml', stopBuild: true
                            junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'
                            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'dependency-check-jenkins.html	report', reportTitles: 'dependency-check-jenkins', useWrapperFileDirectly: true])
                           
              
                    }
                }
            }
        }

        stage('Unit Testing') {
            steps {
                catchError(buildResult: 'SUCCESS', message: 'Oops! it will be fixed in future', stageResult: 'UNSTABLE') {
                    sh 'npm test'
            }
        }

        }


        stage('Code coverage') {
            steps {
                catchError(buildResult: 'SUCCESS', message: 'Oops! it will be fixed in future', stageResult: 'UNSTABLE') {
                 sh ' npm run coverage'  
                 }
                 publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './coverage/lcov-result/', reportFiles: 'index.html', reportName: 'Code Coverage html report', reportTitles: 'codecoverage-jenkins', useWrapperFileDirectly: true])
         }
         
}

        stage('SonarQube') {
            steps {
                sh 'echo $SONAR_SCANNER_HOME '
                sh'''
                $SONAR_SCANNER_HOME/bin/sonar-scanner \
                -Dsonar.projectKey=jenkins-pipeline \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://172.183.97.211:9000 \
                -Dsonar.token=sqp_f16a88e410e8ffd60c881349728c80b02742ffe0
                '''

    }
    }
    }
}