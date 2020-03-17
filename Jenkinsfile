		pipeline {
		agent any
		tools {				
				maven "MAVEN_HOME"
			}
		parameters {
			gitParameter branchFilter: 'origin/(.*)', name: 'BRANCH', defaultValue: 'master', type: 'PT_BRANCH'
		  }
		  environment {
				// This can be nexus3 or nexus2
				NEXUS_VERSION = "nexus3"
				// This can be http or https
				NEXUS_PROTOCOL = "http"
				// Where your Nexus is running
				NEXUS_URL = "34.93.149.51:8081"
				// Repository where we will upload the artifact
				NEXUS_REPOSITORY = "DevopsSnapshot"
				// Jenkins credential id to authenticate to Nexus OSS
				NEXUS_CREDENTIAL_ID = "Nexus"
			}
			
		stages {
		stage ('clean up workspace') {
			steps {
				cleanWs()
				}
				}	
		stage('pull') {
			steps { 
			git branch: "${params.BRANCH}", credentialsId: 'PradeepGithub', url: 'https://github.com/PradeepDrall/SampleApplication.git'
			echo 'pull completed'
		}
		}
		stage ('Compile Stage') {
			steps {
				bat 'mvn clean compile'
			}
		}
		stage ('Testing Stage') {
		steps {
				bat 'mvn test'
			}
			}
		stage ('Install Stage') {
		steps {
			bat 'mvn clean install'
			
			}
		  }
		  stage('Sonarqube') {
			environment {
				scannerHome = tool 'sonar_scanner'
			}
			steps {
				withSonarQubeEnv('SonarQube8.1') {
				bat "${scannerHome}/bin/sonar-scanner.bat"
				bat 'mvn clean package sonar:sonar'
				}
			}
		}
				stage("Sonar Quality Gate") {
					steps {
					  timeout(time: 1, unit: 'HOURS') {
						waitForQualityGate abortPipeline: true
					  }
					}
				  }
				  
		  stage("publish to nexus") {
					steps {
						script {
							// Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is  included and installed peline-utility-steps plugins
							pom = readMavenPom file: "pom.xml";
							// Find built artifact under target folder
							filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
							// Print some info from the artifact found
							echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
							// Extract the path from the File found
							artifactPath = filesByGlob[0].path;
							// Assign to a boolean response verifying If the artifact name exists
							artifactExists = fileExists artifactPath;
							if(artifactExists) {
								echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
								nexusArtifactUploader(
									nexusVersion: NEXUS_VERSION,
									protocol: NEXUS_PROTOCOL,
									nexusUrl: NEXUS_URL,
									groupId: pom.groupId,
									version: pom.version,
									repository: NEXUS_REPOSITORY,
									credentialsId: NEXUS_CREDENTIAL_ID,
									artifacts: [
										// Artifact generated such as .jar, .ear and .war files.
										[artifactId: pom.artifactId,
										classifier: '',
										file: artifactPath,
										type: pom.packaging],
										// Lets upload the pom.xml file for additional information for Transitive dependencies
										[artifactId: pom.artifactId,
										classifier: '',
										file: "pom.xml",
										type: "pom"]
									]
								);
							} else {
								error "*** File: ${artifactPath}, could not be found";
							}
						}
					}
				}
		stage ('Email notification') {
			  steps {
		emailext (
			subject: "Job '${env.JOB_NAME} ${env.BUILD_NUMBER}'",
			body: """<p>Check console output at <a href="${env.BUILD_URL}">${env.JOB_NAME}</a></p>""",
			to: "pradeep.kumar162@wipro.com",
			from: "drallpradeep@gmail.com"
		)
		}
		}		
			}
		}
