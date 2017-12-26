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
			def proc = bat (script:'''"C:\\Program Files (x86)\\nuget.exe "  restore " C:\\Program Files (x86)\\Jenkins\\jobs\\New_notification_test\\branches\\master\\workspace\\C#\\StoreApp.sln"''', returnStatus: true)
			if (proc == 0){
				echo "Restore Nuget Packges";
			}
			else {
				emailext attachLog: true, body:" Nuget Restore falhou favor verificar", subject: "Restore", to: "aline.campos@ventron.com.br";
				currentBuild.result = "FAILURE";
				error 'Problema no restore do nuget packge';
			}
		}	

        stage("Build"){
			def msbuild = tool name: 'MsBuild fw4.0', type: 'hudson.plugins.msbuild.MsBuildInstallation';
			def proc = bat (script:'''"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\MSBuild.exe"   "C:\\Program Files (x86)\\Jenkins\\jobs\\New_notification_test\\branches\\master\\workspace\\C#\\StoreApp.sln"''', returnStatus: true)
			echo "Build Execution";
			if (proc == 0){
				echo "Build execution";
				emailext attachLog: true, body:"Build e Commit feito com sussesso", subject: "Build", to: "aline.campos@ventron.com.br";
			}
			else {
				emailext attachLog: true, body:"Erro no build, por favor verifique o log e reverta o commite caso necessario", subject: "Build", to: EMAIL_TO;
				currentBuild.result = "FAILURE";
				error 'Build Solution';
			}
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
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BRANCH_NAME})"
 
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
 
}
