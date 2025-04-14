pipeline{
    agent{
        label "node"
    }
    tools{
        maven "MAVEN3.9"
        jdk "JDK17"

    }    
    stages{
        stage("Fetch Code"){
            steps{
                git branch: "atom", url: "https://github.com/hkhcoder/vprofile-project.git"
            }
        } 
        
        stage("Build"){
            steps{
                sh "mvn install -DskipTests"
            }
            post{
                success{
                    echo "Archiving artifact"
                    archiveArtifact artifacts: "**/*.war"
                }
            }
        }

        stage("Unit Test"){
            steps{
                sh "mvn test"
            }
        } 

        stage("Checkstyle analysis"){
            steps{
                sh "mvn checkstyle:checkstyle "  
            }
        }

        stage("Sonar Code Analysis"){
            environment{
                scannerHome = tool "sonar6.2"
            }
            steps{
                withSonarQubeEnv("sonarserver"){
                sh '''
                    ${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \

                   -Dsonar.projectName=vprofile \

                   -Dsonar.projectVersion=1.0 \

                   -Dsonar.sources=src/ \

                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \

                   -Dsonar.junit.reportsPath=target/surefire-reports/ \

                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \

                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''

              }
                
            }
        }

        stage("Quality gates"){
            steps{
                timeout(time: 1, unit: "HOURS"){
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("UploadArtifact"){
            steps{
                nexusArtifactUploader{
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '172.31.18.28:8081',
                    groupId: 'QA',
                    version: '${env.BUILD_ID}-${env.BUILD_TIMESTAMP}',
                    repository: 'vprofile-repo',
                    credentialsId: 'nexuslogin',
                    artifacts: [
                        [artifactId: 'vproapp', file: 'target/vprofile-v2.war', type: 'war']
                    ]
                }
            }
        }
            
    }
}
    