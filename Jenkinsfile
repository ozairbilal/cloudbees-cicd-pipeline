def props
def jiraKey
def visionEnvironmentId
def xlDeployUser
def visionPackageId
def result 
def releaseType
def releaseEnv 
pipeline {
		options {
			buildDiscarder logRotator(numToKeepStr: '7')
			disableConcurrentBuilds()
		}
		environment{
			// Jira Credentials for Creation and updation of tickets 
			JIRA_CREDENTIALS=credentials('OzairJiraCredentials')
		}
		agent{
			// TODO: Finalize the labels 
			label "master"
		}
		stages {
			stage("Init"){
				steps {
					script{
						 props = readProperties interpolate: true, file: "system.properties"
						// TODO: Uncomment
						// visionEnvironmentId = "Environments/Vision/${props.environment}/WAS/${BRANCH_NAME}/${BRANCH_NAME}"
						// TODO: Remove
						 visionEnvironmentId = "Environments/Vision/${props.environment}/WAS/Staging-Main/Staging-Main"
						 visionPackageId: "Applications/Visioncore/${props.vision}"
						 xlDeployUser = "scmuser"
						 result = props.releaseType.toLowerCase().split("-")
						releaseType = result[0]
						releaseEnv = result[1]
					}
				}
			}

			stage("JIRA"){
				/*agent {
					label "master"
				}*/
				when {
					anyOf{
					branch "Staging-Canary"
					branch "Staging-Main" 
					branch "Pretest-Canary" 
					branch "Pretest-Main" 
					}
				}
				steps{
					script{
						 jiraKey = jiraCreate(props.vision, props.releaseType.toLowerCase())
					}
				}
			} 
			stage("deploy") {
			 /*   agent {
					label "master"
				}*/
				steps {
					script{
					   /* sh "curl ${props.NexusURL}/vision/${props.vision}/vision-${props.vision}.dar -o vision-${props.vision}.dar"
						script {
							try {
								xldPublishPackage serverCredentials: xlDeployUser, darPath: "vision-${props.vision}.dar"
							} catch(e) {
								echo "The package already exists"
							}
						}*/
						
						if (BRANCH_NAME == "Staging-Canary")
						{
								//xldDeploy environmentId: visionEnvironmentId, packageId: visionPackageId, serverCredentials: xlDeployUser
								echo "Deployed to ${BRANCH_NAME}  Environment"
						}
						
						if (BRANCH_NAME == "Pretest-Canary" || BRANCH_NAME == "Pretest-Main" )
						{
								//xldDeploy environmentId: visionEnvironmentId, packageId: visionPackageId, serverCredentials: xlDeployUser
								//echo result.data.toString()
								echo "Deployed to ${BRANCH_NAME} Environment"
						}
					}
				}
				post{
					success{
						script{
							postDeploymentJiraSuccess(jiraKey, BRANCH_NAME, releaseType)
						}
					}
					 failure{
						script{
							postDeploymentJiraFailure(jiraKey, BRANCH_NAME, releaseType, releaseEnv)
						}
					 }
				}
			}
			
			/*stage("AT") {
				when {
					branch "QMG2"    
				}
				parallel {
					stage("Machine142") {
					   
						steps {
							echo 'uftScenarioLoad archiveTestResultsMode: "ALWAYS_ARCHIVE_TEST_REPORT", testPaths: "my-tests"'
						}
					}
					stage("Machine246") {
					   
						steps {
							echo 'uftScenarioLoad archiveTestResultsMode: "ALWAYS_ARCHIVE_TEST_REPORT", testPaths: "my-tests"'
						}
					}
					stage("Machine74") {
					  
						steps {
							echo 'uftScenarioLoad archiveTestResultsMode: "ALWAYS_ARCHIVE_TEST_REPORT", testPaths: "my-tests"'
						}
					}
					stage("Machine96") {
						
						steps {
							 echo 'uftScenarioLoad archiveTestResultsMode: "ALWAYS_ARCHIVE_TEST_REPORT", testPaths: "my-tests"'
						}
					}
					stage("Machine95") {
					  
						steps {
							//uftScenarioLoad archiveTestResultsMode: "ALWAYS_ARCHIVE_TEST_REPORT", testPaths: "C:\\AFT\\Scripts\\Vision\\BC\\BC_Script"
							bat "java -cp C:\\AFT\\Scripts\\Vision\\VBSFiles-AutoExecution\\; UpdateVBS C:\\AFT\\Scripts\\Vision\\VBSFiles-AutoExecution\\ExecutionController.VBS 11 11 C:\\AFT\\Scripts\\Vision\\BC\\BC_Script"
							bat "cscript C:/AFT/Scripts/Vision/VBSFiles-AutoExecution/ExecutionController.VBS"
						}
					}
				}
			}
			 
			stage("PublishResults") {
				when {
					branch "QMG2"    
				} 
				steps {
					//uftScenarioLoad archiveTestResultsMode: "ALWAYS_ARCHIVE_TEST_REPORT", testPaths: "C:\\AFT\\Scripts\\E2EReport\\E2EReport_Script"
					bat "java -cp C:\\AFT\\Scripts\\Vision\\VBSFiles-AutoExecution\\; UpdateVBS C:\\AFT\\Scripts\\Vision\\VBSFiles-AutoExecution\\ExecutionController.VBS 11 11 C:\\AFT\\Scripts\\E2EReport\\E2EReport_Script"
					bat "cscript C:/AFT/Scripts/Vision/VBSFiles-AutoExecution/ExecutionController.VBS"
					
					bat "xcopy C:\\AFT\\Results\\E2EResults.html  %cd%\\ /y"
					publishConfluence siteName:'emaratech.atlassian.net' , spaceName:'IM',parentId:354354214 ,buildIfUnstable:true, pageName:'Automation Testing Reports', editorList:[confluenceAppendPage(generator:confluenceText('Please see AT attached report'))] , fileSet: 'E2EResults.html'
				}
			}
		   stage("PT") {
				agent {
					// TODO: Finalize the labels 
					label "performance"
				}
				when {
					branch "Staging-Canary"    
				} 
				steps {
					script {
						try {
							loadRunnerTest archiveTestResultsMode: 'PUBLISH_HTML_REPORT', displayController: 'true', fsTimeout: '900', testPaths: '''C:\\Temp\\Scenario.lrs'''
						} catch(e) {}
					}
								   
				   sleep(5)
				   bat """
				   cd C:\\tmp
				   mkdir ${BUILD_NUMBER}
				   """
				   powershell "Copy-Item \"*\\LRR\\reportToPublish.html\" -Destination \"C:\\tmp\\${BUILD_NUMBER}\" -Recurse"
				   powershell "Copy-Item \"*\\LRR\\reportToPublish\" -Destination \"C:\\tmp\\${BUILD_NUMBER}\" -Recurse"
				   bat """
				   Set Zip="C:\\Program Files\\7-Zip\\7z.exe\"
				   %zip% a -tzip "C:\\tmp\\${BUILD_NUMBER}.zip" "C:\\tmp\\${BUILD_NUMBER}" -mx5
				   """
				   bat "xcopy C:\\tmp\\${BUILD_NUMBER}.zip  %cd%\\"
				   publishConfluence siteName:'emaratech.atlassian.net' , spaceName:'IM',parentId:369230625 , buildIfUnstable:true, pageName:'Release ${BUILD_NUMBER}', editorList:[confluenceAppendPage(generator:confluenceText('Please see PT attached report'))] ,fileSet: '${BUILD_NUMBER}.zip' 
				}
			}
*/
    }
}
