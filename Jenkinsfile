pipeline{
    agent any
	
	tools{
	    maven "MAVEN3.9"
		jdk "JDK17"
	}
	
	environment{
	    registryCredential: "ecr:us-east-1:awscreds"
		imageName: "951401132355.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg"
		vprofileRegistry: "https://951401132355.dkr.ecr.us-east-1.amazonaws.com"
		cluster: "vprofile"
		service: "vprofileappsvc"
	}
	
	stages{
	    stage("fetch code"){
		    steps{
			    git branch: "atom", url: "http://github.com/Parulsingh87/vprofile.git"
			}
		}
		
		stage("Build"){
		    steps{
			    sh "mvn install -DskipTests"
			}
			
			post{
			    success{
				    echo "Archiving atifact"
					archiveArtifacts artifacts: "**/*.war"
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
			    sh "mvn checkstyle:checkstyle"
			}
		}
		
		stage("Sonar code analysis"){
		    environment{
			    scannerHome = "sonar6.2"
			}
			steps{
			    withSonarQubeEnv{
				    sh '''
					    ${scannerHome}/bin/sonar-scanner -Dsonar.projectKey = vprofile \
						-DsonarprojectName = vprofile \
						-DprojectVersion = 1.0 \
						-Dsonar.sources = src/ \
						-Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                                                -Dsonar.junit.reportsPath=target/surefire-reports/ \
                                                -Dsonar.jacoco.reportsPath=target/jacoco.exec \
						-Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
					'''
				}      
			}
		}
		
		stage("Quality Gate"){
		    steps{
			    timeout(time: 1, unit: "Hours"){
				    waitForQualityGate abortPipeline: true
				}
			}
		}
		
		stage("Build Image"){
		    steps{
			    script{
				    dockerImage = docker.build(imageName + ":$BUILD_NUMBER", "./Docker-files-app/multistage/")
				}
			}
		}
		
		stage("Upload Image"){
		    steps{
			    script{
				    docker.withRegistry(vprofileRegistry, registryCredential)
					dockerImage.push("$BUILD_NUMBER")
					dockerImage.push("latest")
				}
			}
		}
		
		stage("Upload to ECS"){
		    steps{
			    withAWS(credentials: "awscreds", region: "us-east-1)
				sh "aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment"
			}    
		}
		
		
	}
}
