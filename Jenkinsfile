pipeline {
    agent any

    // tools {
    //     nodejs 'Node 24.4.0'
    // }

    // environment{
    //       SONAR_SCANNER_HOME = tool 'SonarScanner';
    // }

    stages {

        // stage('Installing Dependencies') {
        //     steps {
        //         sh 'npm install --no-audit'
        //     }
        // }

        // stage('Dependency Scanning') {
        //     parallel {
        //         stage('NPM Dependency Audit') {
        //             steps {
        //                 sh '''
        //                     npm audit --audit-level=critical || true
        //                     echo "NPM Audit completed"
        //                 '''
        //             }
        //         }

//                 stage('OWASP Dependency Check') {
//                     steps {
//                         dependencyCheck additionalArguments: '''
//                             --scan ./ \
//                             --out ./ \
//                             --format ALL \
//                             --prettyPrint
//                         ''', odcInstallation: 'OWASP-depend-check-12'
//                             dependencyCheckPublisher failedTotalCritical: 1, pattern: '**/dependency-check-report.xml', stopBuild: true
//                             junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'
//                             publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'dependency-check-jenkins.html	report', reportTitles: 'dependency-check-jenkins', useWrapperFileDirectly: true])
                           
              
//                     }
//                 }
//             }
//         }

//         stage('Unit Testing') {
//             steps {
//                 catchError(buildResult: 'SUCCESS', message: 'Oops! it will be fixed in future', stageResult: 'UNSTABLE') {
//                     sh 'npm test'
//             }
//         }

//         }


//         stage('Code coverage') {
//             steps {
//                 catchError(buildResult: 'SUCCESS', message: 'Oops! it will be fixed in future', stageResult: 'UNSTABLE') {
//                  sh ' npm run coverage'  
//                  }
//                  publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './coverage/lcov-result/', reportFiles: 'index.html', reportName: 'Code Coverage html report', reportTitles: 'codecoverage-jenkins', useWrapperFileDirectly: true])
//          }
         
// }

//         stage('SonarQube') {
//             steps {
                
//                     withSonarQubeEnv('SonarQube'){
//                         sh 'echo $SONAR_SCANNER_HOME '
//                         sh'''
//                             $SONAR_SCANNER_HOME/bin/sonar-scanner \
//                             -Dsonar.projectKey=jenkins-pipeline \
//                             -Dsonar.sources=. \
//                             -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info \
//                         '''
//                     }
//                }    

//             }

        stage('Dockerbuild'){
            steps{
                sh 'printenv'                    //prints all the env variables accesed
                sh ' docker build -t dockerimage:$GIT_COMMIT . '
            }

        }

        stage('Trivy Vulnerability Scanner Docker image'){
            steps{
                sh '''
                   trivy image dockerimage:$GIT_COMMIT \
                        --severity LOW,MEDIUM,HIGH \
                        --exit-code 0 \
                        --quiet \
                        --format json -o trivy-imageMEDIUM-results.json

                    trivy image dockerimage:$GIT_COMMIT \
                        --severity CRITICAL \
                        --exit-code 1 \
                        --quiet \
                        --format json -o trivy-imageCRITICAL-results.json 
                    '''       
            }

            post{
                always{
                    sh '''
                        trivy convert \
                            --format template \
                            --template @/usr/local/share/trivy/templates/html.tpl \
                            --output trivy-imageMEDIUM-results.html \
                            trivy-imageMEDIUM-results.json
                        '''


                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'trivy-imageMEDIUM-results.html', reportName: 'trivy-imageMEDIUM-results.htmlreport', reportTitles: 'trivy-imageMEDIUM-results.html', useWrapperFileDirectly: true])
                }
            }


        }

        stage('Docker Push'){
            when {
                branch 'feature/*'
                }
            steps{
                withDockerRegistry(credentialsId:'DockerHub', url: ""){
                    sh 'docker tag dockerimage:$GIT_COMMIT varshithag30/dockerimage:$GIT_COMMIT'
                    sh 'docker login'
                    sh 'docker push varshithag30/dockerimage:$GIT_COMMIT'
                }
            }
        }

         stage('Deploy to EC2'){
            steps{
                    sshagent(['AWS-SSH private key dev deploy']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ubuntu@54.160.155.240 \\
                                "sudo docker run --name solar-system \\
                                    -e MONGO_URI='mongodb+srv://supercluster.d83jj.mongodb.net/superData' \\
                                    -e MONGO_USERNAME='superuser' \\
                                    -e MONGO_PASSWORD='SuperPassword' \\
                                    -p 3000:3000 -d varshithag30/dockerimage:$GIT_COMMIT"
                        """
                        } 
                }
        }
    }
}


    

    


