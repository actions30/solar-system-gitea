pipeline {
    agent any

    tools {
        nodejs 'Node 24.4.0'
    }

    environment{
          SONAR_SCANNER_HOME = tool 'SonarQube';
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
                timeout(time: 60, unit: 'SECONDS'){
                    withSonarQubeEnv('SonarQube'){
                        sh 'echo $SONAR_SCANNER_HOME '
                        sh'''
                            $SONAR_SCANNER_HOME/bin/sonar-scanner \
                            -Dsonar.projectKey=jenkins-pipeline \
                            -Dsonar.sources=. \
                            -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info \
                        '''
                    }
                    waitForQualityGate abortPipeline:true
               }    

            }
        }
    }
}