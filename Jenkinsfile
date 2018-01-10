#!groovy
import groovy.json.JsonOutput
import java.util.Optional
import hudson.tasks.test.AbstractTestResultAction
import hudson.model.Actionable
import hudson.tasks.junit.CaseResult

node {
    try {
        stage('Checkout') {
            checkout scm
        }

        stage("Restore") {
			def proc = bat (script:'''"C:\\Program Files (x86)\\nuget.exe "  restore " "\"${WORKSPACE}\\**.sln"''', returnStatus: true)
		}"D:\Test_cleanUp\API.V1\MedgrupoAPI.sln"
        stage("Build"){
			def msbuild = tool name: 'MsBuild fw4.0', type: 'hudson.plugins.msbuild.MsBuildInstallation';
			def proc = bat (script:'''"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\MSBuild.exe"   "\"${WORKSPACE}\\**.sln"''', returnStatus: true)
			echo "Build Execution";
		}
		stage("DLL") {
      String DLLS = "E:\\ScriptsJenkins\\Scripts\\git_scripts\\DLL_separator.ps1";
			def power = bat (script: 'powershell "'+DLLS+'"', returnStatus: true)
		}		

  } catch (e) {
    // If there was an exception thrown, the build failed
    currentBuild.result = "FAILED"
    throw e
  } finally {
    // Success or failure, always send notifications
    notifyBuild(currentBuild.result)
  }
}
 
def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'
 
  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
  def summary = "${subject}: Job rodou usando a (${env.BRANCH_NAME})"
  def details = """<p>STARTED: Job  ${env.JOB_NAME} Build number: [${env.BUILD_NUMBER}] </p>
  <p>Branch: ${env.BRANCH_NAME}</p>
  <p>Check console output at the attachments</p>"""
 
  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }
 
  // Send notifications
  slackSend (color: colorCode, message: summary)
    //email
  emailext (
	  mimeType: 'text/html',
	  attachLog: true,
      subject: subject,
      body: details,
      to: "aline.campos@ventron.com.br"
    ) 
}
