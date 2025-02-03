    pipeline {
        tools {
            maven 'm398'
            jdk 'jdk11'
        }
       
        agent any
    
        stages {
            stage('Declarative CheckOut') {
                steps {
                    sh "echo Checking Out code"
                    checkout scm
                }
            }
            stage('Compile') {
                steps {
                    sh "mvn clean compile"
                }
            }
            stage('Unit Test') {
                steps {
                    sh "mvn test"
                }
            }
            stage('Publish Test Result') {
                steps {
                    junit '**/surefire-reports/*.xml'
                }
            }
            stage('Code Quality Check') {
                steps {
                    withSonarQubeEnv('sonar'){
                        sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=akhanal77_cicd'
                       // some block
                    }
                }
            }
             stage('Package Application') {
                 steps {
                     sh "mvn package -DskipTests"
                 }
             }
             stage('Generic Artificatory') {
                 steps {
                 rtUpload (
                         serverId: 'artifactory',
                         spec: '''{
                             "files": [
                                 {
                                 "pattern": "target/ci-pipeline-pragra-0.0.1.jar",
                                 "target": "libs-snapshot-local/pragra-ci-demo/${BUILD_NUMBER}"
                                 },
                                 {
                                 "pattern": "pom.xml",
                                 "target": "libs-snapshot-local/pragra-ci-demo/${BUILD_NUMBER}"
                                 }
                            ]
                        }''',
                   
                         // Optional - Associate the uploaded files with the following custom build name and build number,
                         // as build artifacts.
                         // If not set, the files will be associated with the default build name and build number (i.e the
                         // the Jenkins job name and number).
                         buildName: "${JOB_NAME}",
                         buildNumber: "${BUILD_NUMBER}"
                 )
                 }
             }
   //          stage('Promote Build') {
   //              steps {
   //                  withCredentials([usernameColonPassword(credentialsId: 'artificatory', variable: 'logindata')]) {
   //                   sh 'curl -u${logindata} -X PUT "http://localhost:8081/artifactory/api/storage/libs-snapshot-local/pragra-ci-demo/${BUILD_NUMBER}/ci-pipeline-pragra-0.0.1.jar?properties=Promoted=Yes"'
   //               }
   //            }
   //         }
             stage('Publish to Artifactory') {
                 steps {
                     rtMavenResolver (
                         id: 'resolver1',
                         serverId: 'artifactory',
                         releaseRepo: 'libs-release-local',
                         snapshotRepo: 'libs-snapshot-local'
                     ) 

                     rtMavenDeployer (
                         id: 'deployer1',
                         serverId: 'artifactory',
                         releaseRepo: 'libs-release-local',
                         snapshotRepo: 'libs-snapshot-local'
                     )
                  rtMavenRun (
                         tool:'m398',
                         pom: 'pom.xml',
                         goals: 'install',
                         resolverId: 'resolver1',
                         deployerId: 'deployer1'
                     )
                 }
             }

        }

        post {
            always {
                echo "Looks good"
            }
        }
    }
